# TestingPermissionTool.tsx — 测试权限工具

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/testing/TestingPermissionTool.tsx`
- **类型**: TypeScript/React 测试工具模块
- **功能**: 测试专用工具，调用时总是弹出权限对话框

## 核心内容详解

### 导出常量

```typescript
const NAME = 'TestingPermission'
```

### 输入 Schema

```typescript
{}  // 空对象，无需输入参数
```

### 输出

```typescript
string  // 成功消息
```

### 主要方法

| 方法 | 说明 |
|------|------|
| `checkPermissions()` | 总是返回 `ask` 行为 |
| `call()` | 返回成功消息 |
| `mapToolResultToToolResultBlockParam()` | 格式化结果为工具结果块 |

### 权限行为

```typescript
async checkPermissions() {
  return {
    behavior: 'ask' as const,
    message: `Run test?`
  }
}
```

### 功能开关

```typescript
isEnabled() {
  return "production" === 'test'  // 仅在测试环境启用
}
```

### UI 渲染

所有 UI 渲染方法返回 `null`：
- `renderToolUseMessage()`
- `renderToolUseProgressMessage()`
- `renderToolUseQueuedMessage()`
- `renderToolUseRejectedMessage()`
- `renderToolResultMessage()`
- `renderToolUseErrorMessage()`

## 设计要点

### 测试专用
- 仅在测试环境启用（`process.env.NODE_ENV === 'test'`）
- 用于端到端测试权限系统

### 总是询问
- `checkPermissions()` 总是返回 `behavior: 'ask'`
- 触发权限对话框

### 安全特性
- 并发安全：返回 `true`
- 只读：返回 `true`

## 与其他文件的关系

### 依赖
- `zod/v4` — Schema 验证
- `../../Tool.js` — 工具基类

### 使用场景
- 测试权限对话框显示
- 端到端测试权限流程
- 验证权限系统行为
