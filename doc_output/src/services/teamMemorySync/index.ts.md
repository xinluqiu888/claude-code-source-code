# index.ts — 团队记忆同步服务

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/teamMemorySync/index.ts`
- **作用域**: 团队记忆文件的同步管理
- **主要导出**:
  - `createSyncState`: 创建同步状态
  - `isTeamMemorySyncAvailable`: 检查同步是否可用
  - `pullTeamMemory`: 拉取团队记忆
  - `pushTeamMemory`: 推送团队记忆
  - `hashContent`: 计算内容哈希
  - `batchDeltaByBytes`: 按字节分批

## 功能概述

同步团队记忆文件（存储在 `.claude/team-memory/`）与服务器 API。团队记忆按仓库范围（由 git remote hash 标识）并在所有认证的组织成员间共享。

## 核心内容详解

### 配置常量

```typescript
const TEAM_MEMORY_SYNC_TIMEOUT_MS = 30_000
const MAX_FILE_SIZE_BYTES = 250_000      // 每文件 250KB
const MAX_PUT_BODY_BYTES = 200_000       // PUT 请求体 200KB
const MAX_RETRIES = 3
const MAX_CONFLICT_RETRIES = 2
```

### 同步状态

```typescript
export type SyncState = {
  lastKnownChecksum: string | null        // 最后已知的服务器校验和
  serverChecksums: Map<string, string>   // 服务器每个键的内容哈希
  serverMaxEntries: number | null        // 服务器强制条目上限
}
```

### 核心函数

#### `pullTeamMemory(state, options?)`
从服务器拉取团队记忆并写入本地目录：
1. 检查 OAuth 和 GitHub 仓库
2. 获取服务器数据（支持 ETag 缓存）
3. 更新 `serverChecksums`
4. 写入远程条目到本地
5. 清理记忆文件缓存

**参数**:
- `skipEtagCache`: 跳过 ETag 缓存

**返回**:
- `success`: 是否成功
- `filesWritten`: 实际写入的文件数
- `entryCount`: 服务器返回的条目数
- `notModified`: 304 未修改

#### `pushTeamMemory(state)`
推送本地团队记忆到服务器：
1. 读取本地文件（扫描密钥）
2. 计算本地哈希
3. 计算 delta（与 `serverChecksums` 对比）
4. 分批上传（按字节）
5. 处理 412 冲突（刷新 `serverChecksums` 重试）
6. 更新 `serverChecksums` 和 `lastKnownChecksum`

**冲突处理**:
- 本地编辑优先于服务器版本
- 最大 2 次冲突重试
- 使用 `GET ?view=hashes` 低成本刷新校验和

#### `readLocalTeamMemory(maxEntries)`
读取本地团队记忆文件：
- 递归遍历目录
- 跳过超过 250KB 的文件
- **PSR M22174**: 扫描密钥，包含密钥的文件跳过不上传
- 返回条目和跳过的密钥文件列表

#### `writeRemoteEntriesToLocal(entries)`
写入远程条目到本地：
- 验证路径（防止目录遍历）
- 跳过内容已匹配的文件
- 并行写入
- 返回实际写入的文件数

#### `fetchTeamMemory(state, repoSlug, etag?)`
带重试的获取：
- 支持 304 Not Modified
- 支持 404 空数据
- 更新 `lastKnownChecksum`

#### `fetchTeamMemoryHashes(state, repoSlug)`
轻量级哈希获取（`GET ?view=hashes`）：
- 仅获取元数据和条目校验和
- 用于冲突解决

#### `uploadTeamMemory(state, repoSlug, entries, ifMatchChecksum?)`
上传条目：
- 支持 `If-Match` 条件请求
- 处理 412 Precondition Failed
- 处理结构化 413（条目过多）

#### `batchDeltaByBytes(delta)`
将 delta 分批以保持在 `MAX_PUT_BODY_BYTES` 内：
- 贪婪装箱排序键
- 确定性批次

### 辅助函数

#### `hashContent(content)`
计算 `sha256:<hex>` 格式内容哈希

#### `isUsingOAuth()`
检查是否使用第一方 OAuth（需要 `user:inference` 和 `user:profile` scope）

## 设计要点

1. **增量上传**: 仅上传哈希不同的条目
2. **乐观锁定**: 使用 ETag/If-Match 防止冲突
3. **密钥扫描**: 上传前扫描密钥（PSR M22174）
4. **分批上传**: 大 delta 分成多个 PUT 请求
5. **本地优先**: 冲突时本地编辑优先
6. **状态隔离**: `SyncState` 对象传递避免模块级状态

## 与其他文件的关系

- **types.ts**: 类型定义和 Zod Schema
- **secretScanner.ts**: 密钥扫描
- **watcher.ts**: 文件监视和自动推送
- **teamMemPaths.ts**: 团队记忆路径

## 注意事项

1. **同步语义**:
   - Pull：服务器内容覆盖本地
   - Push：仅上传变化的键（服务器 upsert）
   - 删除不传播

2. **限制**:
   - 每文件 250KB
   - PUT 体 200KB（自动分批）
   - 条目数由服务器动态控制

3. **OAuth 要求**: 需要 `user:inference` 和 `user:profile` scope
