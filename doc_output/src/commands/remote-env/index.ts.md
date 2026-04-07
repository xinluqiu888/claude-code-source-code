# remote-env/index.ts

## 文件描述
Remote Env 命令配置 - 配置远程环境

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | remote-env |
| 描述 | Configure the default remote environment for teleport sessions |
| 启用条件 | isClaudeAISubscriber() && isPolicyAllowed('allow_remote_sessions') |
| 隐藏条件 | 同上 |

## 核心内容

### 命令配置
```typescript
export default {
  type: 'local-jsx',
  name: 'remote-env',
  description: 'Configure the default remote environment for teleport sessions',
  isEnabled: () =>
    isClaudeAISubscriber() && isPolicyAllowed('allow_remote_sessions'),
  get isHidden() {
    return !isClaudeAISubscriber() || !isPolicyAllowed('allow_remote_sessions');
  },
  load: () => import('./remote-env.js'),
} satisfies Command
```

## 设计点

1. **订阅者专用**：需要 Claude AI 订阅
2. **策略限制**：需要远程会话权限
3. **条件隐藏**：不满足条件时隐藏
4. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `isClaudeAISubscriber` 从 `../../utils/auth.js`
- 导入 `isPolicyAllowed` 从 `../../services/policyLimits/index.js`

## 注意事项

- 配置远程会话默认环境
- 需要适当权限
