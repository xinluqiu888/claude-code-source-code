# rate-limit-options/index.ts

## 文件描述
Rate Limit Options 命令配置 - 显示速率限制选项

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | rate-limit-options |
| 描述 | Show options when rate limit is reached |
| 隐藏 | true（内部使用） |
| 启用条件 | isClaudeAISubscriber() |

## 核心内容

### 命令配置
```typescript
const rateLimitOptions = {
  type: 'local-jsx',
  name: 'rate-limit-options',
  description: 'Show options when rate limit is reached',
  isEnabled: () => isClaudeAISubscriber(),
  isHidden: true,
  load: () => import('./rate-limit-options.js'),
} satisfies Command
```

## 设计点

1. **内部命令**：隐藏，不显示在帮助中
2. **订阅者专用**：仅 Claude AI 订阅者可用
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `isClaudeAISubscriber` 从 `../../utils/auth.js`

## 注意事项

- 内部使用命令
- 不显示在帮助列表中
- 到达速率限制时触发
