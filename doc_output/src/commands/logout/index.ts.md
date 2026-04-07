# logout/index.ts

## 文件描述
Logout 命令配置 - 登出 Anthropic 账户

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | logout |
| 描述 | Sign out from your Anthropic account |
| 启用条件 | !DISABLE_LOGOUT_COMMAND |

## 核心内容

### 命令配置
```typescript
export default {
  type: 'local-jsx',
  name: 'logout',
  description: 'Sign out from your Anthropic account',
  isEnabled: () => !isEnvTruthy(process.env.DISABLE_LOGOUT_COMMAND),
  load: () => import('./logout.js'),
} satisfies Command
```

## 设计点

1. **条件启用**：可通过环境变量禁用
2. **清理彻底**：清除缓存和凭据
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `isEnvTruthy` 从 `../../utils/envUtils.js`

## 注意事项

- 清除所有认证信息
- 刷新 GrowthBook
- 优雅关闭
