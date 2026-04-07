# remote-setup/index.ts

## 文件描述
Remote Setup 命令配置 - Web 设置

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | web-setup |
| 描述 | Setup Claude Code on the web (requires connecting your GitHub account) |
| 可用性 | claude-ai |
| 启用条件 | tengu_cobalt_lantern feature && allow_remote_sessions |

## 核心内容

### 命令配置
```typescript
const web = {
  type: 'local-jsx',
  name: 'web-setup',
  description: 'Setup Claude Code on the web (requires connecting your GitHub account)',
  availability: ['claude-ai'],
  isEnabled: () =>
    getFeatureValue_CACHED_MAY_BE_STALE('tengu_cobalt_lantern', false) &&
    isPolicyAllowed('allow_remote_sessions'),
  get isHidden() {
    return !isPolicyAllowed('allow_remote_sessions');
  },
  load: () => import('./remote-setup.js'),
} satisfies Command
```

### 原命令名
- 原名为 'web'，已重命名为 'web-setup'

## 设计点

1. **功能标志控制**：通过 GrowthBook 控制
2. **策略限制**：需要远程会话权限
3. **环境限制**：仅 claude-ai 可用
4. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `getFeatureValue_CACHED_MAY_BE_STALE` 从 `../../services/analytics/growthbook.js`
- 导入 `isPolicyAllowed` 从 `../../services/policyLimits/index.js`

## 注意事项

- 需要 GitHub 账户连接
- 需要适当权限
- 功能可能不稳定
