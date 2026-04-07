# caches.ts — 会话缓存清理工具

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/clear/caches.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 144 行 |
| 主要职责 | 提供会话级缓存清理功能，支持选择性保留后台任务的代理状态 |

## 功能概述

该文件实现了 `/clear` 命令的缓存清理子模块。它清理各种与会话相关的缓存数据，但保持最小的导入依赖以支持启动时的快速加载。支持保留特定代理ID的状态（用于后台任务）。

## 核心内容详解

### 主要函数

**clearSessionCaches**
```typescript
export function clearSessionCaches(
  preservedAgentIds: ReadonlySet<string> = new Set(),
): void
```

清理的缓存类型：
- 用户/系统/Git 上下文缓存
- 文件建议缓存
- 命令/技能缓存
- 提示缓存断点检测状态
- 最后发送日期
- 后压缩清理
- 图片路径缓存
- 会话入口缓存
- LSP 诊断状态
- Magic Docs 追踪
- WebFetch URL 缓存
- 等等...

### 保留逻辑

当 `preservedAgentIds` 非空时：
- 不清除提示缓存断点检测状态
- 不清除挂起的回调
- 不清除 dump 状态
- 选择性清除调用的技能

## 设计要点

1. **最小导入**：文件顶部导入保持最小以优化启动性能
2. **动态导入**：许多清理操作使用动态导入以支持代码分割
3. **条件清理**：根据是否有保留的代理ID调整清理策略
4. **懒加载友好**：模块可被懒加载以减少启动开销

## 与其他文件的关系

- **clear.ts**: 调用 `clearSessionCaches` 作为清理流程的一部分
- **conversation.ts**: 在清除对话时调用
- **各种服务模块**: 提供各自的清理函数
