# index.ts — copy 命令入口配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/copy/index.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 15 行 |
| 主要职责 | 定义 copy 命令的元数据和懒加载配置 |

## 核心内容详解

### 命令配置

```typescript
const copy = {
  type: 'local-jsx',
  name: 'copy',
  description: "Copy Claude's last response to clipboard (or /copy N for the Nth-latest)",
  load: () => import('./copy.js'),
} satisfies Command
```

### 关键特性

| 属性 | 值 | 说明 |
|------|-----|------|
| `type` | `'local-jsx'` | 本地 JSX 组件命令 |
| `name` | `'copy'` | 命令名 |
| `description` | 复制最后响应 | 清晰的描述 |

### 注释说明

文件注释说明：
- 实现从 `copy.tsx` 懒加载
- 目的是减少启动时间

## 设计要点

1. **轻量级入口**：仅包含元数据，实现懒加载
2. **清晰描述**：描述中包含用法提示（`/copy N`）

## 与其他文件的关系

- **copy.tsx**: 实际的命令实现
