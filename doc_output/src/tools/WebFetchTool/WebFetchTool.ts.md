# WebFetchTool.ts — 网页获取工具

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/WebFetchTool/WebFetchTool.ts`
- **类型**: TypeScript 工具模块
- **功能**: 获取网页内容并处理

## 功能概述

WebFetchTool 从指定 URL 获取内容，将 HTML 转换为 Markdown，然后使用 AI 模型处理内容并返回结果。

## 核心内容详解

### 输入 Schema

```typescript
{
  url: string     // 要获取内容的 URL
  prompt: string  // 在获取内容上运行的提示
}
```

### 输出 Schema

```typescript
{
  bytes: number      // 内容大小（字节）
  code: number       // HTTP 响应码
  codeText: string   // HTTP 响应文本
  result: string     // 处理后的结果
  durationMs: number // 耗时（毫秒）
  url: string        // 实际获取的 URL
}
```

### 主要方法

| 方法 | 说明 |
|------|------|
| `checkPermissions()` | 权限检查，支持预批准域名 |
| `validateInput()` | 验证 URL 格式 |
| `call()` | 执行获取和处理操作 |
| `mapToolResultToToolResultBlockParam()` | 格式化结果为工具结果块 |

### 核心逻辑

1. **URL 验证**: 检查 URL 格式和长度
2. **权限检查**: 检查预批准域名或权限规则
3. **内容获取**: 获取 URL 内容，支持重定向处理
4. **内容处理**: 使用 Haiku 模型处理内容
5. **结果返回**: 返回处理后的结果和元数据

### 重定向处理

- 检测重定向到不同主机的情况
- 返回特殊格式的重定向消息
- 要求用户使用新的 URL 重新调用

## 设计要点

### 预批准域名
- 使用 `isPreapprovedHost()` 检查域名
- 预批准域名跳过部分内容限制

### 权限系统
- 支持 `allow`/`deny`/`ask` 规则
- 基于域名的规则匹配

### 缓存机制
- 15 分钟缓存（utils.ts 中实现）
- 提高重复访问的性能

### 并发安全
- `isConcurrencySafe(): true`
- `isReadOnly(): true`

## 与其他文件的关系

### 依赖
- `./preapproved.ts` — 预批准域名列表
- `./utils.ts` — 内容获取和处理工具函数
- `./UI.tsx` — UI 渲染组件
- `../../utils/permissions/permissions.js` — 权限检查

### 使用场景
- 获取网页文档
- 分析网页内容
- 获取代码库信息
