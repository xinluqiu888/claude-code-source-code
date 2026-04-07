# datadog.ts — Datadog 日志服务

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/analytics/datadog.ts`
- **所属模块**: Analytics Service
- **功能类型**: 遥测后端集成

## 功能概述

该模块实现与 Datadog 日志服务的集成，将分析事件批量发送到 Datadog 的 HTTP 日志摄入端点。支持批量处理、自动刷新和事件过滤。

## 核心内容详解

### 配置常量

```typescript
DATADOG_LOGS_ENDPOINT = 'https://http-intake.logs.us5.datadoghq.com/api/v2/logs'
DATADOG_CLIENT_TOKEN = 'pubbbf48e6d78dae54bceaa4acf463299bf'
DEFAULT_FLUSH_INTERVAL_MS = 15000  // 15秒刷新间隔
MAX_BATCH_SIZE = 100               // 最大批次大小
NETWORK_TIMEOUT_MS = 5000          // 网络超时
```

### 允许的事件列表

`DATADOG_ALLOWED_EVENTS` 定义了允许发送到 Datadog 的事件白名单（63 个事件），包括：
- Chrome Bridge 相关事件（连接、工具调用等）
- Tengu API 事件（成功、错误、取消等）
- OAuth 事件（成功、错误、令牌刷新等）
- 团队内存同步事件

### 主要函数

#### `initializeDatadog(): Promise<boolean>`
初始化 Datadog 客户端（使用 memoize 缓存）。
- 检查分析是否被禁用
- 返回初始化状态

#### `trackDatadogEvent(eventName, properties): Promise<void>`
发送事件到 Datadog 的核心函数。

**数据处理流程：**
1. 仅在生产环境发送
2. 第三方提供商（非 First Party）直接返回
3. 获取并丰富事件元数据
4. 数据规范化：
   - MCP 工具名规范化为 "mcp"（降基）
   - 模型名规范化为短名称
   - 版本号截断（去除时间戳和 SHA）
   - 状态码转换为 http_status 和 http_status_range

#### `shutdownDatadog(): Promise<void>`
优雅关闭 Datadog 客户端。
- 清除刷新定时器
- 刷新剩余日志批次

### 批次处理机制

```
事件入队 → 批次未满 → 定时刷新
    ↓           ↓
批次已满 → 立即刷新 → HTTP POST
```

### 标签字段

`TAG_FIELDS` 定义了可用于 Datadog 搜索和过滤的高基数字段：
- 平台相关：arch, platform, version
- 模型相关：model, provider
- 用户相关：userType, subscriptionType, userBucket
- 错误相关：errorType, http_status

### 用户分桶

`getUserBucket()` — 基于用户 ID 哈希计算分桶（0-29），用于：
- 估算受影响用户数量而不直接计数用户 ID
- 在保持用户隐私的同时减少基数字

## 设计要点

1. **白名单机制** — 仅允许预定义的事件列表，防止敏感事件外泄
2. **数据规范化** — MCP 工具名、模型名、版本号均规范化处理
3. **批量发送** — 减少网络请求，提高性能
4. **异步刷新** — 不阻塞主流程
5. **仅生产环境** — 开发环境事件不发送

## 与其他文件的关系

- **被调用**: `sink.ts` 的路由逻辑
- **依赖**: `metadata.ts` 获取事件元数据
- **关联**: `config.ts` 检查分析是否启用

## 注意事项

- 第三方提供商（Bedrock/Vertex/Foundry）的事件不会发送到 Datadog
- 事件名称必须在 `DATADOG_ALLOWED_EVENTS` 白名单中
- 版本号会被截断以降基（例如：2.0.53-dev.20251124.t173302.sha526cc6a → 2.0.53-dev.20251124）
- 使用 memoize 缓存初始化结果和用户分桶
