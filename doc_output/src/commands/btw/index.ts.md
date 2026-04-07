# index.ts — btw 命令入口配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/btw/index.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 13 行 |
| 主要职责 | 定义 btw 命令的元数据，实现即时旁支查询功能 |

## 核心内容详解

### 命令配置

```typescript
const btw = {
  type: 'local-jsx',
  name: 'btw',
  description: 'Ask a quick side question without interrupting the main conversation',
  immediate: true,
  argumentHint: '<question>',
  load: () => import('./btw.js'),
} satisfies Command
```

### 关键特性

| 属性 | 值 | 说明 |
|------|-----|------|
| `type` | `'local-jsx'` | 本地 JSX 组件命令 |
| `name` | `'btw'` | 命令名（By The Way 的缩写） |
| `immediate` | `true` | 立即执行，不进入主消息循环 |
| `argumentHint` | `'<question>'` | 必需的问题参数 |

## 设计要点

**即时模式 (`immediate: true`)**
- 命令不进入主消息循环
- 立即执行并返回结果
- 不影响主对话的上下文

## 与其他文件的关系

- **btw.tsx**: 实际的命令实现
- **commands.ts**: 命令类型定义
