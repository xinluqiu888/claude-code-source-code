# login/index.ts

## 文件描述
Login 命令配置 - 登录 Anthropic 账户

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | login |
| 描述 | 动态描述（根据认证状态） |
| 启用条件 | !DISABLE_LOGIN_COMMAND |

## 核心内容

### 命令配置
```typescript
export default () => ({
  type: 'local-jsx',
  name: 'login',
  description: hasAnthropicApiKeyAuth()
    ? 'Switch Anthropic accounts'
    : 'Sign in with your Anthropic account',
  isEnabled: () => !isEnvTruthy(process.env.DISABLE_LOGIN_COMMAND),
  load: () => import('./login.js'),
}) satisfies Command
```

### 动态描述
- 已认证：显示"切换账户"
- 未认证：显示"登录"

## 设计点

1. **动态描述**：根据认证状态调整
2. **条件启用**：可通过环境变量禁用
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `hasAnthropicApiKeyAuth` 从 `../../utils/auth.js`
- 导入 `isEnvTruthy` 从 `../../utils/envUtils.js`

## 注意事项

- 支持账户切换
- 需要浏览器认证
