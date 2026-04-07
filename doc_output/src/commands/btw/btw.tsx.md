# btw.tsx — 侧边问题快速询问功能

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/btw/btw.tsx` |
| 文件类型 | TypeScript React (.tsx) |
| 主要职责 | 提供快速侧边问题询问功能，不打断主对话流程 |

## 功能概述

该文件实现了 `/btw` (By The Way) 命令，允许用户在不中断主对话的情况下快速询问一个旁支问题。这是一个轻量级的查询功能，使用当前对话上下文但不会影响主会话的消息历史。

## 核心内容详解

### 主要组件

**BtwSideQuestion 组件**
- 接收 `question`、`context` 和 `onDone` 属性
- 显示加载状态（使用 SpinnerGlyph）
- 展示问题的回答结果
- 支持滚动查看长回答

**call 函数**
- 接收用户输入的问题
- 调用 `runSideQuestion` 执行旁支查询
- 使用 `CacheSafeParams` 复用主会话的缓存以提高效率

### 核心特性

1. **缓存优化**：通过 `getLastCacheSafeParams` 复用主会话的 systemPrompt 和上下文
2. **隔离执行**：旁支查询不会影响主对话的消息历史
3. **快捷键支持**：支持上/下箭头滚动，空格/回车/Esc 关闭

## 设计要点

1. **即时响应**：作为即时命令，不进入主消息循环
2. **缓存命中**：优先使用主会话的缓存参数以提高效率
3. **用户友好**：提供清晰的加载状态和滚动提示

## 与其他文件的关系

- **sideQuestion.ts**: 提供 `runSideQuestion` 执行逻辑
- **forkedAgent.ts**: 提供缓存参数管理
- **index.ts**: 命令注册配置
