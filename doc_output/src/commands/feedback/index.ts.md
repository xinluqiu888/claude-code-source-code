# feedback/index.ts

## 文件描述
Feedback 命令配置 - 提交反馈

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | feedback |
| 别名 | bug |
| 描述 | Submit feedback about Claude Code |
| 参数提示 | [report] |
| 复杂启用条件 | 多条件判断 |

## 核心内容

### 命令配置
```typescript
const feedback = {
  aliases: ['bug'],
  type: 'local-jsx',
  name: 'feedback',
  description: 'Submit feedback about Claude Code',
  argumentHint: '[report]',
  isEnabled: () => !(
    isEnvTruthy(process.env.CLAUDE_CODE_USE_BEDROCK) ||
    isEnvTruthy(process.env.CLAUDE_CODE_USE_VERTEX) ||
    isEnvTruthy(process.env.CLAUDE_CODE_USE_FOUNDRY) ||
    isEnvTruthy(process.env.DISABLE_FEEDBACK_COMMAND) ||
    isEnvTruthy(process.env.DISABLE_BUG_COMMAND) ||
    isEssentialTrafficOnly() ||
    process.env.USER_TYPE === 'ant' ||
    !isPolicyAllowed('allow_product_feedback')
  ),
  load: () => import('./feedback.js'),
} satisfies Command
```

### 禁用条件
- 使用 Bedrock/Vertex/Foundry
- 禁用反馈命令的环境变量
- 仅基本流量模式
- Ant 内部用户
- 策略不允许

## 设计点

1. **多别名支持**：bug 别名便于报告问题
2. **复杂启用**：多条件判断
3. **隐私保护**：尊重各种隐私设置
4. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `isEnvTruthy` 从 `../../utils/envUtils.js`
- 导入 `isPolicyAllowed` 从 `../../services/policyLimits/index.js`

## 注意事项

- 受多种环境因素影响
- 隐私优先设计
- 企业环境可能禁用
