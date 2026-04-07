# index.ts — cost 命令入口配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/cost/index.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 23 行 |
| 主要职责 | 定义 cost 命令的元数据和可见性控制 |

## 核心内容详解

### 命令配置

```typescript
const cost = {
  type: 'local',
  name: 'cost',
  description: 'Show the total cost and duration of the current session',
  get isHidden() {
    // 订阅用户隐藏（但 Ant 除外）
    if (process.env.USER_TYPE === 'ant') {
      return false
    }
    return isClaudeAISubscriber()
  },
  supportsNonInteractive: true,
  load: () => import('./cost.js'),
} satisfies Command
```

### 关键特性

| 属性 | 值 | 说明 |
|------|-----|------|
| `type` | `'local'` | 本地命令（非 JSX） |
| `name` | `'cost'` | 命令名 |
| `isHidden` | getter | 订阅用户隐藏（Ant 除外） |
| `supportsNonInteractive` | `true` | 支持非交互式会话 |

### 可见性逻辑

```
isHidden = USER_TYPE === 'ant' ? false : isClaudeAISubscriber()
```

- **Ant 员工**：始终可见（可以看到成本细分）
- **订阅用户**：隐藏（不需要关心成本）
- **非订阅用户**：可见

## 设计要点

1. **用户感知**：根据用户类型和订阅状态智能控制可见性
2. **Ant 例外**：内部员工始终可以查看成本
3. **订阅者友好**：订阅用户不需要关心成本，命令对他们隐藏

## 与其他文件的关系

- **cost.ts**: 实际的命令实现
- **auth.ts**: 提供 `isClaudeAISubscriber` 函数
