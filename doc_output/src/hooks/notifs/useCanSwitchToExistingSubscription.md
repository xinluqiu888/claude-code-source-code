# useCanSwitchToExistingSubscription.tsx — 订阅切换提示

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/notifs/useCanSwitchToExistingSubscription.tsx`
- **类型**: React Hook (TSX)
- **导出函数**: `useCanSwitchToExistingSubscription`
- **依赖**: useStartupNotification, OAuth, auth, config

## 功能概述

本 Hook 检查用户是否拥有 Claude Max/Pro 订阅但未登录使用，如果是则显示提示通知，引导用户使用 `/login` 激活订阅。

## 核心内容详解

### 显示限制

```typescript
const MAX_SHOW_COUNT = 3
```

最多显示 3 次，超过后不再显示。

### 检查流程

```typescript
async function checkSubscription() {
  // 1. 检查显示次数
  if ((getGlobalConfig().subscriptionNoticeCount ?? 0) >= MAX_SHOW_COUNT) {
    return null
  }
  
  // 2. 获取订阅类型
  const subscriptionType = await getExistingClaudeSubscription()
  if (subscriptionType === null) return null
  
  // 3. 增加计数
  saveGlobalConfig(current => ({
    ...current,
    subscriptionNoticeCount: (current.subscriptionNoticeCount ?? 0) + 1,
  }))
  
  // 4. 记录分析
  logEvent('tengu_switch_to_subscription_notice_shown', {})
  
  // 5. 返回通知
  return {
    key: 'switch-to-subscription',
    jsx: <Text color="suggestion">
      Use your existing Claude {subscriptionType} plan with Claude Code
      <Text color="text" dimColor> · /login to activate</Text>
    </Text>,
    priority: 'low',
  }
}
```

### 订阅检查

```typescript
async function getExistingClaudeSubscription(): Promise<'Max' | 'Pro' | null> {
  // 已使用订阅认证则无需切换
  if (isClaudeAISubscriber()) return null
  
  const profile = await getOauthProfileFromApiKey()
  if (!profile) return null
  
  if (profile.account.has_claude_max) return 'Max'
  if (profile.account.has_claude_pro) return 'Pro'
  
  return null
}
```

## 设计要点

### 1. 次数限制

使用全局配置存储显示次数，避免过度打扰。

### 2. 启动时检查

利用 `useStartupNotification` 在挂载时检查，避免运行时重复检查。

### 3. 异步检查

需要网络请求获取 OAuth 资料，使用异步计算函数。

### 4. 美观提示

使用 JSX 格式显示，包含样式和命令提示。

### 5. 分析追踪

记录事件用于分析提示效果。

## 与其他文件的关系

- **useStartupNotification.ts**: 基础 Hook
- **getOauthProfile.ts**: 获取 OAuth 资料
- **auth.ts**: 检查订阅状态
- **config.ts**: 读写全局配置

## 注意事项

1. **隐私考虑**: 需要网络请求获取用户资料
2. **次数持久化**: 计数保存在全局配置中，跨会话有效
3. **仅 API Key 用户**: 已订阅用户不会看到此提示
4. **网络依赖**: 无网络时检查会静默失败
