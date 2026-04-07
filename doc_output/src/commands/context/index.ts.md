# index.ts — context 命令入口配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/context/index.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 24 行 |
| 主要职责 | 定义 context 命令的交互式和非交互式版本 |

## 核心内容详解

### 交互式版本

```typescript
export const context: Command = {
  name: 'context',
  description: 'Visualize current context usage as a colored grid',
  isEnabled: () => !getIsNonInteractiveSession(),
  type: 'local-jsx',
  load: () => import('./context.js'),
}
```

### 非交互式版本

```typescript
export const contextNonInteractive: Command = {
  type: 'local',
  name: 'context',
  supportsNonInteractive: true,
  description: 'Show current context usage',
  get isHidden() { return !getIsNonInteractiveSession() },
  isEnabled() { return getIsNonInteractiveSession() },
  load: () => import('./context-noninteractive.js'),
}
```

### 版本切换逻辑

| 版本 | 条件 | 行为 |
|------|------|------|
| 交互式 | 非交互式会话 | 禁用 |
| 非交互式 | 交互式会话 | 隐藏 |
| 非交互式 | 非交互式会话 | 启用 |

## 设计要点

1. **双版本设计**：为不同会话类型提供最适合的界面
2. **互斥启用**：两个版本不会同时启用
3. **条件隐藏**：非交互式版本在交互式会话中隐藏

## 与其他文件的关系

- **context.tsx**: 交互式实现
- **context-noninteractive.ts**: 非交互式实现
