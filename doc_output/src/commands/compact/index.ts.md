# index.ts — compact 命令入口配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/compact/index.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 15 行 |
| 主要职责 | 定义 compact 命令的元数据和启用条件 |

## 核心内容详解

### 命令配置

```typescript
const compact = {
  type: 'local',
  name: 'compact',
  description: 'Clear conversation history but keep a summary in context...',
  isEnabled: () => !isEnvTruthy(process.env.DISABLE_COMPACT),
  supportsNonInteractive: true,
  argumentHint: '<optional custom summarization instructions>',
  load: () => import('./compact.js'),
} satisfies Command
```

### 关键特性

| 属性 | 值 | 说明 |
|------|-----|------|
| `type` | `'local'` | 本地命令（非 JSX） |
| `name` | `'compact'` | 命令名 |
| `isEnabled` | 函数 | 检查 DISABLE_COMPACT 环境变量 |
| `supportsNonInteractive` | `true` | 支持非交互式会话 |
| `argumentHint` | 可选指令 | 支持自定义摘要指令 |

## 设计要点

1. **环境变量控制**：可通过 `DISABLE_COMPACT` 禁用压缩功能
2. **非交互式支持**：CI/CD 环境也可使用
3. **可选参数**：支持自定义摘要生成指令

## 与其他文件的关系

- **compact.ts**: 实际的命令实现
- **envUtils.ts**: 提供 `isEnvTruthy` 函数
