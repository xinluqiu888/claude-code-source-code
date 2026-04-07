# postCompactCleanup.ts — 压缩后清理

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/compact/postCompactCleanup.ts`
- **作用域**: 压缩后的状态和缓存清理
- **主要导出**:
  - `runPostCompactCleanup`: 执行压缩后清理

## 功能概述

在压缩（自动或手动）完成后执行各种清理操作，释放被失效追踪结构占用的内存。区分主线程和子代理的清理策略，避免破坏共享的模块级状态。

## 核心内容详解

### 清理内容

```typescript
export function runPostCompactCleanup(querySource?: QuerySource): void {
  // 1. 重置微压缩状态
  resetMicrocompactState()
  
  // 2. 上下文折叠重置（主线程）
  if (feature('CONTEXT_COLLAPSE') && isMainThreadCompact) {
    resetContextCollapse()
  }
  
  // 3. 用户上下文缓存清理（主线程）
  if (isMainThreadCompact) {
    getUserContext.cache.clear?.()
    resetGetMemoryFilesCache('compact')
  }
  
  // 4. 系统提示章节清理
  clearSystemPromptSections()
  
  // 5. 分类器批准清理
  clearClassifierApprovals()
  
  // 6. 推测性检查清理
  clearSpeculativeChecks()
  
  // 7. Beta 追踪状态清理
  clearBetaTracingState()
  
  // 8. 会话消息缓存清理
  clearSessionMessagesCache()
}
```

### 主线程检测

```typescript
const isMainThreadCompact =
  querySource === undefined ||
  querySource.startsWith('repl_main_thread') ||
  querySource === 'sdk'
```

### 子代理保护

子代理（`agent:*`）与主线程在同一进程中运行，共享模块级状态：
- 上下文折叠存储
- memory file 缓存
- `getUserContext` 缓存

子代理压缩时不应重置这些状态，否则会破坏主线程的状态。

## 设计要点

1. **线程感知**: 区分主线程和子代理的清理范围
2. **选择性清理**: 技能内容（`invoked_skills`）不清理，以便后续压缩附件使用
3. **Feature 保护**: 使用 `feature()` 检查保护特性相关的清理

## 与其他文件的关系

- **autoCompact.ts**: 调用 `runPostCompactCleanup`
- **compact.ts**: 调用 `runPostCompactCleanup`
- **contextCollapse/index.ts**: 提供上下文折叠重置
- **sessionStorage.ts**: 提供会话消息缓存清理

## 注意事项

1. **querySource 参数**: 所有压缩调用者应传递此参数以确保正确清理
2. **共享状态风险**: 子代理共享模块级状态，需要特殊保护
3. **技能内容保留**: 故意不调用 `resetSentSkillNames()`
