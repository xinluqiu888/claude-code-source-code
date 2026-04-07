# watcher.ts — 团队记忆文件监视器

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/teamMemorySync/watcher.ts`
- **作用域**: 团队记忆目录的文件监视和自动推送
- **主要导出**:
  - `startTeamMemoryWatcher`: 启动团队记忆监视器
  - `stopTeamMemoryWatcher`: 停止监视器
  - `notifyTeamMemoryWrite`: 通知团队记忆写入
  - `_resetWatcherStateForTesting`: 测试用状态重置

## 功能概述

监视团队记忆目录的变更，在文件修改时触发防抖推送。启动时执行初始拉取，然后使用 `fs.watch` 监视目录。

## 核心内容详解

### 配置常量

```typescript
const DEBOUNCE_MS = 2000  // 最后变更后等待 2 秒再推送
```

### 监视器状态

```typescript
let watcher: FSWatcher | null = null
let debounceTimer: ReturnType<typeof setTimeout> | null = null
let pushInProgress = false
let hasPendingChanges = false
let currentPushPromise: Promise<void> | null = null
let watcherStarted = false
let pushSuppressedReason: string | null = null  // 永久失败时抑制推送
let syncState: SyncState | null = null
```

### 核心函数

#### `startTeamMemoryWatcher()`
启动团队记忆同步系统：

**启动条件**:
- `TEAMMEM` 构建标志开启
- 团队记忆启用
- OAuth 可用
- 有 github.com remote

**流程**:
1. 检查所有条件
2. 创建同步状态
3. 执行初始拉取
4. 启动文件监视器
5. 记录遥测

#### `startFileWatcher(teamDir)`
启动文件监视：
- 使用 `fs.watch({recursive: true})`
- 递归监视子目录
- macOS FSEvents O(1) fds
- Linux inotify O(subdirs)

**事件处理**:
- `filename === null`: 直接调度推送
- 抑制状态：stat 检查是否为 unlink（ENOENT），是则清除抑制
- 正常状态：调度推送

**为什么选择 `fs.watch` 而非 chokidar**:
- chokidar 4+ 放弃 fsevents
- Bun 的 `fs.watch` 回退使用 kqueue
- kqueue 需要每个监视文件一个 fd
- 500+ 团队记忆文件 = 500+ 永久持有的 fds

#### `schedulePush()`
防抖推送调度：
1. 检查抑制状态
2. 设置待处理标记
3. 清除现有定时器
4. 设置新定时器（2 秒）
5. 推送进行中时，完成后重新调度

#### `executePush()`
执行推送：
1. 设置推送中标记
2. 调用 `pushTeamMemory`
3. 成功：清除待处理标记
4. 失败：检查是否永久失败，设置抑制
5. 记录遥测

#### `isPermanentFailure(result)`
判断是否为永久失败（无需用户操作无法自愈）：
- `no_oauth`: 无 OAuth
- `no_repo`: 无仓库
- 4xx（除 409/429）：客户端错误

**非永久失败**:
- 409: 冲突，刷新后可重试
- 429: 速率限制，稍后重试

#### `stopTeamMemoryWatcher()`
停止监视器：
1. 清除防抖定时器
2. 关闭 watcher
3. 等待进行中的推送
4. 如有待处理变更，尝试最后一次推送

**注意**: 在 2 秒优雅关闭预算内运行，推送是尽力而为。

#### `notifyTeamMemoryWrite()`
显式通知团队记忆写入（例如从 PostToolUse hooks）：
- 在 watcher 启动的同一 tick 写入的文件可能不触发事件
- 一些平台会合并快速连续写入
- 显式通知确保推送被调度

### 抑制机制

当推送因永久失败原因失败时，设置 `pushSuppressedReason`：
- 防止其他会话写入共享团队目录导致的无限重试循环
- 仅通过 unlink 清除（条目过多恢复路径）
- `no_oauth` 用户的抑制持续到会话重启

## 设计要点

1. **防抖推送**: 2 秒防抖避免频繁推送
2. **递归监视**: 支持子目录
3. **抑制机制**: 永久失败时停止重试
4. **平台优化**: macOS FSEvents O(1)，Linux inotify O(subdirs)
5. **显式通知**: PostToolUse hooks 显式通知确保不遗漏
6. **优雅关闭**: 2 秒内尽力刷新待处理变更

## 与其他文件的关系

- **index.ts**: 提供 `pullTeamMemory`、`pushTeamMemory`、`createSyncState`
- **teamMemPaths.ts**: 提供 `getTeamMemPath`、`isTeamMemoryEnabled`

## 注意事项

1. **fs.watch 限制**: 不区分 add/change/unlink，都触发 `rename`
2. **平台差异**: macOS FSEvents，Linux inotify
3. **抑制清除**: 仅 unlink 事件清除抑制
4. **测试限制**: `feature('TEAMMEM')` 在 bun test 下为 false
