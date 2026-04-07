# cost/index.ts

## 文件描述
Cost 命令配置 - 显示会话成本

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local |
| 名称 | cost |
| 描述 | Show the total cost and duration of the current session |
| 隐藏条件 | 订阅者隐藏（Ant 用户除外） |
| 支持非交互 | true |

## 核心内容

### 命令配置
```typescript
const cost = {
  type: 'local',
  name: 'cost',
  description: 'Show the total cost and duration of the current session',
  get isHidden() {
    if (process.env.USER_TYPE === 'ant') return false;
    return isClaudeAISubscriber();
  },
  supportsNonInteractive: true,
  load: () => import('./cost.js'),
} satisfies Command
```

### 隐藏逻辑
- Ant 用户始终可见
- 订阅者隐藏（已付费）
- 非订阅者可见成本信息

## 设计点

1. **条件显示**：根据用户类型和订阅状态显示
2. **非交互支持**：支持非交互模式
3. **懒加载**：动态导入实现

## 与其他文件的关系

- 导入 `isClaudeAISubscriber` 从 `../../utils/auth.js`

## 注意事项

- 订阅者看不到成本（已包含在订阅中）
- Ant 内部用户可以看到成本明细
- 支持非交互输出
