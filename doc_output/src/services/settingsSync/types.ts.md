# types.ts — 设置同步类型定义

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/settingsSync/types.ts`
- **作用域**: 用户设置同步 API 的类型和 Zod Schema
- **主要导出**:
  - `UserSyncDataSchema`: 用户同步数据 Zod Schema
  - `UserSyncContentSchema`: 同步内容 Schema
  - `SettingsSyncFetchResult`: 获取结果类型
  - `SettingsSyncUploadResult`: 上传结果类型
  - `SYNC_KEYS`: 同步键常量

## 功能概述

定义用户设置和记忆文件同步的数据结构。基于后端 API 契约，支持用户设置跨 Claude Code 环境同步。

## 核心内容详解

### Zod Schema

#### `UserSyncContentSchema`
内容部分 - 扁平键值存储：
```typescript
export const UserSyncContentSchema = lazySchema(() =>
  z.object({
    entries: z.record(z.string(), z.string()),  // 键：文件路径，值：UTF-8 内容
  }),
)
```

#### `UserSyncDataSchema`
完整 API 响应：
```typescript
export const UserSyncDataSchema = lazySchema(() =>
  z.object({
    userId: z.string(),
    version: z.number(),
    lastModified: z.string(),     // ISO 8601 时间戳
    checksum: z.string(),         // MD5 哈希
    content: UserSyncContentSchema(),
  }),
)
```

### 类型定义

#### `UserSyncData`
用户同步数据类型：
```typescript
export type UserSyncData = z.infer<ReturnType<typeof UserSyncDataSchema>>
```

#### `SettingsSyncFetchResult`
获取操作结果：
```typescript
export type SettingsSyncFetchResult = {
  success: boolean
  data?: UserSyncData
  isEmpty?: boolean    // 404 表示无数据
  error?: string
  skipRetry?: boolean
}
```

#### `SettingsSyncUploadResult`
上传操作结果：
```typescript
export type SettingsSyncUploadResult = {
  success: boolean
  checksum?: string
  lastModified?: string
  error?: string
}
```

### 同步键常量

```typescript
export const SYNC_KEYS = {
  USER_SETTINGS: '~/.claude/settings.json',
  USER_MEMORY: '~/.claude/CLAUDE.md',
  projectSettings: (projectId: string) =>
    `projects/${projectId}/.claude/settings.local.json`,
  projectMemory: (projectId: string) => `projects/${projectId}/CLAUDE.local.md`,
} as const
```

支持的同步文件：
| 键 | 文件 |
|-----|------|
| `USER_SETTINGS` | `~/.claude/settings.json` |
| `USER_MEMORY` | `~/.claude/CLAUDE.md` |
| `projectSettings(projectId)` | `.claude/settings.local.json` |
| `projectMemory(projectId)` | `CLAUDE.local.md` |

## 设计要点

1. **扁平存储**: 使用键值对而非嵌套结构
2. **路径作为键**: 文件路径作为记录键
3. **版本控制**: 包含版本号支持未来扩展
4. **校验和**: MD5 哈希用于完整性验证
5. **时间戳**: ISO 8601 格式的时间戳

## 与其他文件的关系

- **index.ts**: 使用这些类型进行 API 调用和数据处理

## 注意事项

1. **大小限制**: 每文件 500KB（后端限制）
2. **空文件**: 404 响应表示无数据（`isEmpty: true`）
3. **Zod 验证**: 所有 API 响应都通过 Zod Schema 验证
