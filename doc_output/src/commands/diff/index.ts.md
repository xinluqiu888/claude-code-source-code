# index.ts — diff 命令入口配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/diff/index.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 8 行 |
| 主要职责 | 定义 diff 命令的元数据 |

## 核心内容详解

### 命令配置

```typescript
export default {
  type: 'local-jsx',
  name: 'diff',
  description: 'View uncommitted changes and per-turn diffs',
  load: () => import('./diff.js'),
} satisfies Command
```

### 关键特性

| 属性 | 值 | 说明 |
|------|-----|------|
| `type` | `'local-jsx'` | 本地 JSX 组件 |
| `name` | `'diff'` | 命令名 |
| `description` | 查看未提交更改和每轮差异 | 功能描述 |

## 设计要点

1. **简单配置**：无特殊启用条件或别名
2. **清晰描述**：说明支持查看两种类型的差异

## 与其他文件的关系

- **diff.tsx**: 实际的命令实现
