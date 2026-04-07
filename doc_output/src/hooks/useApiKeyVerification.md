# useApiKeyVerification.ts — API 密钥验证 Hook

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/useApiKeyVerification.ts`
- **类型**: React Hook
- **导出函数**: `useApiKeyVerification`
- **导出类型**: `VerificationStatus`, `ApiKeyVerificationResult`

## 功能概述

本 Hook 管理 Claude Code 的 API 密钥验证状态，支持：
1. 多种 API 密钥来源（环境变量、配置文件、apiKeyHelper）
2. 异步验证流程
3. Claude AI 订阅者自动验证通过
4. 错误状态管理

## 核心内容详解

### 验证状态类型

```typescript
type VerificationStatus = 
  | 'loading'   // 验证中
  | 'valid'     // 验证通过
  | 'invalid'   // 密钥无效
  | 'missing'   // 未找到密钥
  | 'error'     // 验证过程出错
```

### 初始化逻辑

**跳过验证的情况**
- Anthropic 认证未启用：`!isAnthropicAuthEnabled()`
- Claude AI 订阅者：`isClaudeAISubscriber()`

**加载状态判断**
```typescript
const { key, source } = getAnthropicApiKeyWithSource({
  skipRetrievingKeyFromApiKeyHelper: true,  // 安全考虑
})
if (key || source === 'apiKeyHelper') {
  return 'loading'
}
```

### 验证流程 (verify 函数)

1. **Claude AI 订阅者检查**
   - 订阅者直接返回 'valid'，不检查密钥

2. **apiKeyHelper 预热**
   ```typescript
   await getApiKeyFromApiKeyHelper(getIsNonInteractiveSession())
   ```
   - 非交互式会话中异步获取密钥
   - 结果缓存供后续使用

3. **密钥获取**
   ```typescript
   const { key: apiKey, source } = getAnthropicApiKeyWithSource()
   ```

4. **密钥验证**
   - 调用 `verifyApiKey(apiKey, false)` 进行服务端验证
   - 返回布尔值表示密钥有效性

5. **错误处理**
   - API 错误（非 401）：返回 'error' 并保存错误信息
   - apiKeyHelper 返回无效密钥：返回 'error'

### 安全考虑

**skipRetrievingKeyFromApiKeyHelper 选项**
- 初始检查时跳过 apiKeyHelper 执行
- 防止在信任对话框显示前执行 settings.json 中的代码
- 避免远程代码执行（RCE）风险

## 设计要点

### 1. 延迟验证策略

- 使用 `useState` 的函数初始化形式避免不必要的重新验证
- 验证结果通过 `reverify` 函数手动刷新

### 2. 多源密钥支持

```typescript
type ApiKeySource = 
  | 'env'           // ANTHROPIC_API_KEY 环境变量
  | 'config'        // ~/.claude/config.json
  | 'apiKeyHelper'  // 外部助手脚本
```

### 3. 错误传播

- 网络错误、服务错误等非 401 错误通过 `error` 字段暴露
- UI 可以显示详细错误信息帮助用户诊断

## 与其他文件的关系

- **auth.ts**: 提供 `getAnthropicApiKeyWithSource`, `getApiKeyFromApiKeyHelper`, `isAnthropicAuthEnabled`, `isClaudeAISubscriber`
- **claude.ts**: 提供 `verifyApiKey` 服务端验证函数
- **state.ts**: 提供 `getIsNonInteractiveSession`

## 注意事项

1. **Claude AI 订阅者**: 完全绕过密钥验证，依赖订阅状态检查
2. **apiKeyHelper**: 支持动态获取密钥（如从密码管理器），但有安全风险
3. **状态持久化**: Hook 返回的状态是瞬时的，应用重启后需要重新验证
4. **错误处理**: API 返回错误但不一定是密钥无效时，同时设置 error 和 status
