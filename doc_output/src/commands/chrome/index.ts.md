# index.ts — chrome 命令入口配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/chrome/index.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 13 行 |
| 主要职责 | 定义 chrome 命令的元数据和启用条件 |

## 核心内容详解

### 命令配置

```typescript
const command: Command = {
  name: 'chrome',
  description: 'Claude in Chrome (Beta) settings',
  availability: ['claude-ai'],
  isEnabled: () => !getIsNonInteractiveSession(),
  type: 'local-jsx',
  load: () => import('./chrome.js'),
}
```

### 关键特性

| 属性 | 值 | 说明 |
|------|-----|------|
| `name` | `'chrome'` | 命令名 |
| `availability` | `['claude-ai']` | 仅在 claude-ai 平台可用 |
| `isEnabled` | 函数 | 非交互式会话时禁用 |
| `type` | `'local-jsx'` | 本地 JSX 组件 |

## 设计要点

1. **平台限制**：仅对 `claude-ai` 平台用户可用
2. **交互式限制**：非交互式会话中禁用
3. **Beta 标注**：描述中标注为 Beta 功能

## 与其他文件的关系

- **chrome.tsx**: 实际的设置界面实现
- **bootstrap/state.ts**: 提供 `getIsNonInteractiveSession`
