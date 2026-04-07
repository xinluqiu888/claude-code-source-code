# consolidationLock.ts — 整合锁文件管理

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/autoDream/consolidationLock.ts`
- **作用域**: 基于文件的锁机制，防止并发整合
- **主要导出**:
  - `readLastConsolidatedAt`: 读取最后整合时间
  - `tryAcquireConsolidationLock`: 尝试获取锁
  - `rollbackConsolidationLock`: 回滚锁状态
  - `listSessionsTouchedSince`: 列出指定时间后访问的会话
  - `recordConsolidation`: 记录手动整合

## 功能概述

使用文件锁机制协调多个 Claude 进程之间的记忆整合。锁文件的 mtime 表示 `lastConsolidatedAt`，内容是持有者的 PID。位于记忆目录内，与记忆文件使用相同的 git-root 键。

## 核心内容详解

### 常量

```typescript
const LOCK_FILE = '.consolidate-lock'
const HOLDER_STALE_MS = 60 * 60 * 1000  // 1小时，即使 PID 存活也视为过期
```

### 锁文件格式

- **路径**: `{autoMemPath}/.consolidate-lock`
- **mtime**: 最后整合时间戳（毫秒）
- **内容**: 持有者进程 PID（字符串）

### 核心函数

#### `readLastConsolidatedAt()`
读取最后整合时间：

**返回值**: `Promise<number>`
- 锁文件存在: 返回 `mtimeMs`
- 锁文件不存在: 返回 `0`

**成本**: 每次调用一次 `stat`

#### `tryAcquireConsolidationLock()`
尝试获取整合锁：

**返回值**: `Promise<number | null>`
- `number`: 获取成功，返回获取前的 mtime（用于回滚）
- `null`: 获取失败（被其他进程持有）

**逻辑**:
1. 读取现有锁文件的 stat 和 PID
2. 检查锁是否过期（超过 HOLDER_STALE_MS）
3. 如果 PID 存活且未过期，返回 null（获取失败）
4. 否则视为死锁，尝试回收
5. 创建记忆目录（如果不存在）
6. 写入当前 PID 到锁文件
7. 重新读取验证（防止竞争条件）
8. 如果验证通过，返回原 mtime

**竞争处理**:
- 两个回收者同时写入，最后一个写入的 PID 获胜
- 失败者通过重读验证发现 PID 不匹配，返回 null

#### `rollbackConsolidationLock(priorMtime)`
回滚锁状态：

**参数**:
- `priorMtime`: 获取锁之前的 mtime

**逻辑**:
- `priorMtime === 0`: 删除锁文件（恢复到无文件状态）
- 其他: 清空文件内容并恢复 mtime

**用途**: 整合失败后回滚时间戳，使时间门再次通过

#### `listSessionsTouchedSince(sinceMs)`
列出指定时间后访问的会话：

**参数**:
- `sinceMs`: 时间戳（毫秒）

**返回值**: `Promise<string[]>` — 会话 ID 数组

**注意**:
- 使用 mtime（最后访问时间），非 birthtime（ext4 上为0）
- 排除当前工作目录下的工作树会话（安全跳过）

#### `recordConsolidation()`
记录手动整合：

**用途**: 手动 `/dream` 命令后调用，更新最后整合时间

**特点**:
- 乐观执行（提示构建时触发，无完成后钩子）
- 尽力而为（失败仅记录调试日志）

## 设计要点

1. **文件锁机制**: 简单可靠的跨进程协调
2. **PID + 超时**: 防止死锁和 PID 重用
3. **竞争安全**: 写入后重新读取验证
4. **回滚支持**: 失败时可恢复状态
5. **git-root 键**: 与记忆文件使用相同的项目标识

## 与其他文件的关系

- **memdir/paths.ts**: 提供 `getAutoMemPath`
- **bootstrap/state.ts**: 提供 `getOriginalCwd`
- **utils/sessionStorage.ts**: 提供 `getProjectDir`
- **utils/listSessionsImpl.ts**: 提供 `listCandidates`
- **utils/genericProcessUtils.ts**: 提供 `isProcessRunning`

## 注意事项

1. **锁文件位置**: 位于记忆目录内，非独立位置
2. **超时机制**: 1小时强制过期，防止僵尸锁
3. **手动触发**: `recordConsolidation` 不支持回滚（无 priorMtime 返回）
4. **错误处理**: 使用 `logForDebugging` 记录失败，不抛出
