# index.ts — effort 命令入口配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/effort/index.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 13 行 |
| 主要职责 | 定义 effort 命令的元数据和即时执行条件 |

## 核心内容详解

### 命令配置

```typescript
export default {
  type: 'local-jsx',
  name: 'effort',
  description: 'Set effort level for model usage',
  argumentHint: '[low|medium|high|max|auto]',
  get immediate() {
    return shouldInferenceConfigCommandBeImmediate()
  },
  load: () => import('./effort.js'),
} satisfies Command
```

### 关键特性

| 属性 | 值 | 说明 |
|------|-----|------|
| `type` | `'local-jsx'` | 本地 JSX 组件 |
| `name` | `'effort'` | 命令名 |
| `argumentHint` | 可选级别 | 提示可用选项 |
| `immediate` | getter | 动态决定是否即时执行 |

### 即时执行条件

```typescript
get immediate() {
  return shouldInferenceConfigCommandBeImmediate()
}
```

`shouldInferenceConfigCommandBeImmediate()` 函数决定配置命令是否应该即时执行，可能基于当前会话状态或设置。

## 设计要点

1. **动态即时性**：根据条件决定是否即时执行
2. **级别提示**：参数提示中列出所有可用级别
3. **配置命令**：作为推理配置命令处理

## 与其他文件的关系

- **effort.tsx**: 实际的命令实现
- **immediateCommand.ts**: 提供 `shouldInferenceConfigCommandBeImmediate`
