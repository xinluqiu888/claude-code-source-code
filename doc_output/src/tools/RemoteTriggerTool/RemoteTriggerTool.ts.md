# RemoteTriggerTool.ts — 远程触发器管理工具

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/RemoteTriggerTool/RemoteTriggerTool.ts`
- **作用**: 通过claude.ai CCR API管理远程Claude Code代理触发器

## 功能概述

该工具提供对claude.ai远程触发器API的封装，支持列出、获取、创建、更新和运行远程触发器。身份验证在进程内自动处理，OAuth token不会暴露到shell。

## 核心内容详解

### 输入Schema

```typescript
z.strictObject({
  action: z.enum(['list', 'get', 'create', 'update', 'run']),
  trigger_id: z
    .string()
    .regex(/^\[\w-\]+$/)
    .optional()
    .describe('Required for get, update, and run'),
  body: z
    .record(z.string(), z.unknown())
    .optional()
    .describe('JSON body for create and update'),
})
```

trigger_id格式：只允许字母、数字、下划线和连字符

### 输出Schema

```typescript
z.object({
  status: z.number(),  // HTTP状态码
  json: z.string(),    // API返回的JSON字符串
})
```

### API端点与操作映射

| 操作 | 方法 | URL |
|------|------|-----|
| list | GET | /v1/code/triggers |
| get | GET | /v1/code/triggers/{trigger_id} |
| create | POST | /v1/code/triggers |
| update | POST | /v1/code/triggers/{trigger_id} |
| run | POST | /v1/code/triggers/{trigger_id}/run |

### 请求头

```typescript
const headers = {
  Authorization: `Bearer ${accessToken}`,
  'Content-Type': 'application/json',
  'anthropic-version': '2023-06-01',
  'anthropic-beta': TRIGGERS_BETA,  // 'ccr-triggers-2026-01-30'
  'x-organization-uuid': orgUUID,
}
```

### 启用条件

```typescript
isEnabled() {
  return (
    getFeatureValue_CACHED_MAY_BE_STALE('tengu_surreal_dali', false) &&
    isPolicyAllowed('allow_remote_sessions')
  )
}
```

需要同时满足：
1. GrowthBook特性开关`tengu_surreal_dali`为true
2. 策略允许`allow_remote_sessions`

## 设计要点

1. **只读检测**: `isReadOnly`根据action判断（list/get为只读）
2. **并发安全**: `isConcurrencySafe() => true`
3. **延迟执行**: `shouldDefer: true`
4. **自动认证**: `checkAndRefreshOAuthTokenIfNeeded()`自动刷新token
5. **超时设置**: 20秒超时
6. **取消支持**: 使用`abortController.signal`支持请求取消

## 与其他文件的关系

- **oauth.ts**: `getOauthConfig()`获取API基础URL
- **growthbook.ts**: 特性开关检查
- **policyLimits/index.ts**: 策略权限检查
- **auth.ts**: OAuth token管理
- **UI.tsx**: 渲染函数

## 注意事项

- 需要用户已登录claude.ai账户
- 需要能解析组织UUID
- trigger_id必须符合`^[\w-]+$`正则格式
