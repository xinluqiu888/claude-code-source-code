# ids.ts — 会话与代理 ID 类型定义

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/types/ids.ts`
- **类型**: TypeScript 模块
- **导出内容**: `SessionId` 类型、`AgentId` 类型、`asSessionId()`、`asAgentId()`、`toAgentId()`
- **依赖关系**: 无外部依赖

## 功能概述

本文件定义了 Claude Code 中核心标识符的品牌类型（Branded Types），用于在编译时区分会话 ID 和代理 ID，防止意外混淆。它提供了类型安全的 ID 创建、转换和验证函数。

## 核心内容详解

### 1. SessionId 类型 (第10行)

会话 ID 类型，唯一标识一个 Claude Code 会话：

```typescript
export type SessionId = string & { readonly __brand: 'SessionId' }
```

**特点**:
- 基于字符串的交集类型
- 带有 `'SessionId'` 品牌标记
- 由 `getSessionId()` 返回

**使用示例**:
```typescript
const sessionId: SessionId = getSessionId()
// 错误：不能将普通字符串赋值给 SessionId
const wrong: SessionId = "some-id"  // Error!
```

### 2. AgentId 类型 (第17行)

代理 ID 类型，唯一标识会话内的子代理：

```typescript
export type AgentId = string & { readonly __brand: 'AgentId' }
```

**特点**:
- 带有 `'AgentId'` 品牌标记
- 由 `createAgentId()` 返回
- 存在时表示上下文是子代理（非主会话）

**使用场景**:
- Agent Tool 创建的子代理
- 本地代理任务
- 队友任务

### 3. asSessionId() 函数 (第23-25行)

将原始字符串转换为 SessionId（强制转换）：

```typescript
export function asSessionId(id: string): SessionId {
  return id as SessionId
}
```

**警告**: 谨慎使用，最好使用 `getSessionId()` 获取

### 4. asAgentId() 函数 (第31-33行)

将原始字符串转换为 AgentId（强制转换）：

```typescript
export function asAgentId(id: string): AgentId {
  return id as AgentId
}
```

**警告**: 谨慎使用，最好使用 `createAgentId()` 创建

### 5. toAgentId() 函数 (第35-44行)

验证并将字符串转换为 AgentId（安全转换）：

```typescript
const AGENT_ID_PATTERN = /^a(?:.+-)?[0-9a-f]{16}$/

export function toAgentId(s: string): AgentId | null {
  return AGENT_ID_PATTERN.test(s) ? (s as AgentId) : null
}
```

**验证模式**:
- 以 `a` 开头
- 可选的 `<label>-` 前缀
- 后跟 16 位十六进制字符
- 示例: `a-abc123`, `a1234567890abcdef`

**返回值**:
- 匹配时返回 `AgentId`
- 不匹配（如队友名称、团队寻址）返回 `null`

## 设计要点

1. **品牌类型模式**: 使用 TypeScript 品牌类型在编译时防止 ID 混用
2. **字符串基础**: ID 本质上是字符串，可以序列化存储
3. **验证模式**: AgentId 有严格的格式验证
4. **向后兼容**: SessionId 目前只有强制转换，无验证

## 使用场景

```typescript
// 区分不同类型的 ID
function processId(id: SessionId | AgentId) {
  // 编译时就知道 ID 类型
}

// 安全转换
const agentId = toAgentId(someString)
if (agentId) {
  // 有效的 AgentId
} else {
  // 可能是队友名称或其他格式
}

// 从存储恢复时使用强制转换
const restoredSessionId = asSessionId(storedValue)
```

## 与其他文件的关系

- **bootstrap/state.ts**: `getSessionId()` 返回 SessionId
- **tasks/**: Agent 相关任务使用 AgentId
- **types/message.ts**: Message 类型包含可选的 agentId

## 注意事项

1. **品牌类型是编译时概念**: 运行时品牌标记不存在，只是类型提示
2. **AgentId 有格式要求**: 不是所有字符串都能成为 AgentId
3. **强制转换要谨慎**: `asSessionId` 和 `asAgentId` 不验证格式
4. **序列化友好**: ID 可以安全地 JSON 序列化

## ID 格式对比

| 类型 | 格式示例 | 创建方式 | 验证 |
|------|----------|----------|------|
| SessionId | `sess_xxx` | `getSessionId()` | 无验证函数 |
| AgentId | `a-abc1234567890` | `createAgentId()` | `toAgentId()` 验证 |
