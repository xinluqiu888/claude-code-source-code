# sessionMemoryUtils.ts — 会话记忆工具函数

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/SessionMemory/sessionMemoryUtils.ts`
- **作用域**: 会话记忆配置管理和状态追踪
- **主要导出**:
  - `getLastSummarizedMessageId`: 获取最后摘要消息 ID
  - `setLastSummarizedMessageId`: 设置最后摘要消息 ID
  - `waitForSessionMemoryExtraction`: 等待提取完成
  - `getSessionMemoryContent`: 获取会话记忆内容
  - `getSessionMemoryConfig`: 获取会话记忆配置
  - `hasMetInitializationThreshold`: 检查是否达到初始化阈值
  - `hasMetUpdateThreshold`: 检查是否达到更新阈值

## 功能概述

提供会话记忆的实用函数，可与主 sessionMemory.ts 分离导入以避免循环依赖。这些函数用于配置管理、状态追踪和内容读取。

## 核心内容详解

### 配置类型

```typescript
export type SessionMemoryConfig = {
  minimumMessageTokensToInit: number    // 初始化所需最小 tokens
  minimumTokensBetweenUpdate: number    // 更新间隔 tokens
  toolCallsBetweenUpdates: number       // 更新间隔工具调用数
}

const DEFAULT_SESSION_MEMORY_CONFIG: SessionMemoryConfig = {
  minimumMessageTokensToInit: 10000,
  minimumTokensBetweenUpdate: 5000,
  toolCallsBetweenUpdates: 3,
}
```

### 状态管理

```typescript
let lastSummarizedMessageId: string | undefined
let extractionStartedAt: number | undefined
let tokensAtLastExtraction = 0
let sessionMemoryInitialized = false
```

### 核心函数

#### 消息 ID 管理

```typescript
export function getLastSummarizedMessageId(): string | undefined
export function setLastSummarizedMessageId(messageId: string | undefined): void
```

用于标记已摘要到会话记忆的消息边界，压缩时使用此 ID 确定保留哪些消息。

#### 提取状态管理

```typescript
export function markExtractionStarted(): void
export function markExtractionCompleted(): void
export async function waitForSessionMemoryExtraction(): Promise<void>
```

`waitForSessionMemoryExtraction`:
- 等待进行中的提取完成
- 15 秒超时
- 1 分钟陈旧的提取会被跳过

#### 内容读取

```typescript
export async function getSessionMemoryContent(): Promise<string | null>
```

从 `~/.claude/projects/<path>/.claude/SESSION_MEMORY.md` 读取会话记忆内容。

#### 配置管理

```typescript
export function getSessionMemoryConfig(): SessionMemoryConfig
export function setSessionMemoryConfig(config: Partial<SessionMemoryConfig>): void
```

#### 阈值检查

```typescript
export function hasMetInitializationThreshold(currentTokenCount: number): boolean
export function hasMetUpdateThreshold(currentTokenCount: number): boolean
```

阈值计算：
- 初始化：`currentTokenCount >= minimumMessageTokensToInit`
- 更新：`currentTokenCount - tokensAtLastExtraction >= minimumTokensBetweenUpdate`

使用与 autocompact 相同的 token 计算方法（input + output + cache tokens）。

#### 其他辅助函数

```typescript
export function recordExtractionTokenCount(currentTokenCount: number): void
export function isSessionMemoryInitialized(): boolean
export function markSessionMemoryInitialized(): void
export function getToolCallsBetweenUpdates(): number
export function resetSessionMemoryState(): void
```

### 超时配置

```typescript
const EXTRACTION_WAIT_TIMEOUT_MS = 15000        // 等待超时：15 秒
const EXTRACTION_STALE_THRESHOLD_MS = 60000     // 陈旧阈值：1 分钟
```

## 设计要点

1. **循环依赖避免**: 独立文件避免与 sessionMemory.ts 的循环依赖
2. **模块级状态**: 闭包内维护状态，测试时可重置
3. **容错等待**: 超时和陈旧检测防止无限等待
4. **配置合并**: 部分配置更新时与默认值合并

## 与其他文件的关系

- **sessionMemory.ts**: 主模块，调用这些工具函数
- **sessionMemoryCompact.ts**: 压缩模块使用这些函数
- **filesystem.ts**: 提供 `getSessionMemoryPath`

## 注意事项

1. **Token 计算**: 使用与 autocompact 相同的 `tokenCountWithEstimation`
2. **状态持久化**: 模块级状态在会话期间保持
3. **测试重置**: `resetSessionMemoryState` 用于测试清理
4. **提取等待**: 压缩前应调用 `waitForSessionMemoryExtraction` 确保数据一致
