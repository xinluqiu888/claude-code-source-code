# types.ts — 团队记忆同步类型定义

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/teamMemorySync/types.ts`
- **作用域**: 团队记忆同步 API 的类型和 Zod Schema
- **主要导出**:
  - `TeamMemoryDataSchema`: 团队记忆数据 Schema
  - `TeamMemoryContentSchema`: 内容 Schema
  - `TeamMemoryTooManyEntriesSchema`: 条目过多错误 Schema
  - `TeamMemorySyncFetchResult`: 获取结果类型
  - `TeamMemorySyncPushResult`: 推送结果类型
  - `TeamMemoryHashesResult`: 哈希结果类型

## 功能概述

定义团队记忆同步 API 的数据结构，基于后端 API 契约（anthropic/anthropic#250711、#283027、#293258）。

## 核心内容详解

### Zod Schema

#### `TeamMemoryContentSchema`
内容部分 - 扁平键值存储：
```typescript
export const TeamMemoryContentSchema = lazySchema(() =>
  z.object({
    entries: z.record(z.string(), z.string()),  // 相对路径 -> 内容
    entryChecksums: z.record(z.string(), z.string()).optional(),  // 条目校验和
  }),
)
```

#### `TeamMemoryDataSchema`
完整 API 响应：
```typescript
export const TeamMemoryDataSchema = lazySchema(() =>
  z.object({
    organizationId: z.string(),
    repo: z.string(),
    version: z.number(),
    lastModified: z.string(),      // ISO 8601
    checksum: z.string(),          // SHA256 带 'sha256:' 前缀
    content: TeamMemoryContentSchema(),
  }),
)
```

#### `TeamMemoryTooManyEntriesSchema`
结构化 413 错误体：
```typescript
export const TeamMemoryTooManyEntriesSchema = lazySchema(() =>
  z.object({
    error: z.object({
      details: z.object({
        error_code: z.literal('team_memory_too_many_entries'),
        max_entries: z.number().int().positive(),
        received_entries: z.number().int().positive(),
      }),
    }),
  }),
)
```

### 类型定义

#### `TeamMemoryData`
团队记忆数据类型推断

#### `TeamMemorySyncFetchResult`
获取操作结果：
```typescript
export type TeamMemorySyncFetchResult = {
  success: boolean
  data?: TeamMemoryData
  isEmpty?: boolean          // 404 无数据
  notModified?: boolean      // 304 ETag 匹配
  checksum?: string
  error?: string
  skipRetry?: boolean
  errorType?: 'auth' | 'timeout' | 'network' | 'parse' | 'unknown'
  httpStatus?: number
}
```

#### `TeamMemoryHashesResult`
轻量级哈希探针结果（`GET ?view=hashes`）：
```typescript
export type TeamMemoryHashesResult = {
  success: boolean
  version?: number
  checksum?: string
  entryChecksums?: Record<string, string>
  error?: string
  errorType?: 'auth' | 'timeout' | 'network' | 'parse' | 'unknown'
  httpStatus?: number
}
```

用于 412 冲突解决期间低成本刷新 `serverChecksums`。

#### `TeamMemorySyncPushResult`
推送操作结果：
```typescript
export type TeamMemorySyncPushResult = {
  success: boolean
  filesUploaded: number
  checksum?: string
  conflict?: boolean         // 412 Precondition Failed
  error?: string
  skippedSecrets?: SkippedSecretFile[]  // PSR M22174
  errorType?: 'auth' | 'timeout' | 'network' | 'conflict' | 'unknown' | 'no_oauth' | 'no_repo'
  httpStatus?: number
}
```

#### `TeamMemorySyncUploadResult`
上传操作结果：
```typescript
export type TeamMemorySyncUploadResult = {
  success: boolean
  checksum?: string
  lastModified?: string
  conflict?: boolean
  error?: string
  errorType?: 'auth' | 'timeout' | 'network' | 'unknown'
  httpStatus?: number
  serverErrorCode?: 'team_memory_too_many_entries'
  serverMaxEntries?: number
  serverReceivedEntries?: number
}
```

#### `SkippedSecretFile`
因包含密钥而跳过的文件：
```typescript
export type SkippedSecretFile = {
  path: string
  ruleId: string    // gitleaks 规则 ID
  label: string     // 人类可读标签
}
```

**注意**: 仅记录规则 ID，不记录密钥值或路径（避免泄露仓库结构）。

## 设计要点

1. **entryChecksums**: 可选，前向兼容旧服务器
2. **结构化 413**: 包含 `error_code` 和 `extra_details`
3. **轻量级探针**: `view=hashes` 仅获取元数据
4. **密钥跳过**: 详细记录跳过的文件类型

## 与其他文件的关系

- **index.ts**: 使用这些类型进行 API 调用

## 注意事项

1. **校验和格式**: `sha256:<hex>`
2. **时间戳格式**: ISO 8601
3. **条目数上限**: 服务器动态调整（GB-tunable）
