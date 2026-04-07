# bridgePointer.ts — 远程控制会话崩溃恢复指针管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/bridge/bridgePointer.ts`
- **文件类型**: TypeScript 模块
- **行数**: 约 211 行
- **主要职责**: 管理桥接会话的崩溃恢复指针文件，支持会话恢复和工作树感知读取

## 功能概述

`bridgePointer.ts` 实现了远程控制会话的崩溃恢复机制。当桥接会话创建时，会立即写入一个指针文件；在会话期间定期刷新；在干净关闭时清除。如果进程异常终止（崩溃、kill -9、终端关闭），指针文件会保留。下次启动时，`claude remote-control` 命令可以检测到此文件并提供恢复选项。

该模块使用文件的 mtime（修改时间）而非嵌入的时间戳来检查新鲜度，因此定期重写相同内容的文件可以刷新过期时钟。这与后端的滚动 `BRIDGE_LAST_POLL_TTL`（4小时）语义相匹配。指针文件按工作目录作用域存储，避免并发桥接会话之间的冲突。

此外，模块还实现了工作树感知读取功能，用于 `--continue` 恢复流程。由于 REPL 桥接可能在工作树变异后写入指针到 `getOriginalCwd()`，而 `claude remote-control --continue` 在 shell CWD 运行，因此需要跨 git 工作树兄弟目录搜索最新指针。

## 核心内容详解

### 常量定义

| 常量 | 值 | 说明 |
|------|-----|------|
| `MAX_WORKTREE_FANOUT` | 50 | 工作树扇出上限，防止并行 stat() 突发和异常配置 |
| `BRIDGE_POINTER_TTL_MS` | 4 * 60 * 60 * 1000 (4小时) | 指针文件有效期 |

### 类型定义

#### `BridgePointer` 类型
```typescript
{
  sessionId: string;      // 会话ID
  environmentId: string;  // 环境ID
  source: 'standalone' | 'repl';  // 来源类型
}
```

使用 Zod 模式进行验证：
- `sessionId`: 字符串，必需
- `environmentId`: 字符串，必需
- `source`: 枚举值（'standalone' 或 'repl'），必需

### 核心函数

#### `getBridgePointerPath(dir: string): string`
生成指定目录的桥接指针文件路径。路径格式：`{projectsDir}/{sanitizedDir}/bridge-pointer.json`

#### `writeBridgePointer(dir: string, pointer: BridgePointer): Promise<void>`
写入指针文件：
1. 获取文件路径
2. 递归创建父目录
3. 将指针数据序列化为 JSON 并写入
4. 使用 `utf8` 编码
5. 错误被捕获并记录到调试日志，不会抛出

**刷新机制**: 使用相同 ID 调用会更新 mtime，作为新鲜度刷新。

#### `readBridgePointer(dir: string): Promise<(BridgePointer & { ageMs: number }) \| null>`
读取指针文件并计算年龄：
1. 获取文件路径
2. 获取文件 stat（mtime）然后读取内容
3. 解析并验证 JSON 模式
4. 计算年龄：`Date.now() - mtimeMs`
5. 检查是否过期（`ageMs > BRIDGE_POINTER_TTL_MS`）
6. 过期或无效时自动删除并返回 null

**注意**: 直接使用 stat 和 read，不进行存在性检查（遵循 CLAUDE.md 的 TOCTOU 规则）。

#### `readBridgePointerAcrossWorktrees(dir: string): Promise<{ pointer, dir } \| null>`
工作树感知读取，用于 `--continue` 恢复：
1. **快速路径**: 首先检查当前目录
2. **扇出路径**: 如果快速路径失败，获取工作树列表
3. 过滤掉当前目录（去重）
4. 并行读取所有候选工作树的指针
5. 选择最新鲜的（ageMs 最小）
6. 返回指针和所在目录，以便调用者在恢复失败时清除正确的文件

**性能考虑**:
- 快速路径仅涉及一次 stat 调用
- 仅在必要时才执行 `git worktree list`
- 并行读取减少延迟
- 扇出上限为 50 个工作树

#### `clearBridgePointer(dir: string): Promise<void>`
删除指针文件：
- 使用 `unlink` 删除文件
- ENOENT 错误被忽略（预期在干净关闭后）
- 其他错误记录到调试日志

### 辅助函数

#### `safeJsonParse(raw: string): unknown`
安全 JSON 解析，解析失败返回 null 而非抛出。

## 设计要点

1. **MTIME 作为新鲜度锚点**: 不依赖嵌入的时间戳，而是通过文件的 mtime 判断新鲜度。这使得定期重写相同内容即可刷新过期时钟。

2. **工作目录作用域**: 指针文件存储在项目目录下（与转录 JSONL 文件并列），避免不同仓库的并发桥接会话冲突。

3. **快速路径优先**: 工作树感知读取首先检查当前目录，覆盖独立桥接和未发生工作树变异的 REPL 桥接场景。

4. **防御性清理**: 无效模式或过期的指针文件会被自动删除，防止重复提示恢复已 GC 的环境。

5. **扇出上限**: 工作树数量超过 50 时放弃扇出搜索，防止路径ological 配置导致的性能问题。

6. **路径规范化**: 使用 `sanitizePath` 处理大小写和分隔符差异，确保 Windows 上 git 输出的路径与存储路径匹配。

7. **最佳努力**: 所有 I/O 操作都是最佳努力，错误被捕获并记录，不会中断流程。

## 与其他文件的关系

| 文件 | 关系描述 |
|------|----------|
| `../utils/getWorktreePathsPortable.js` | 获取可移植的工作树路径列表 |
| `../utils/sessionStoragePortable.js` | 获取项目目录和路径清理 |
| `../utils/slowOperations.js` | 慢速 JSON 操作（parse/stringify） |
| `../utils/errors.js` | ENOENT 错误检测 |
| `../utils/lazySchema.js` | 延迟加载 Zod 模式 |

## 注意事项

1. **TOCTOU 规则**: `readBridgePointer` 中 stat 和 read 是分开的两次系统调用，这是有意的设计，因为 mtime 是返回的数据而非 TOCTOU 防护条件。

2. **4小时 TTL**: 与后端 `BRIDGE_LAST_POLL_TTL` 匹配。如果桥接已轮询 5+ 小时后崩溃，只要刷新在窗口内运行，指针仍被视为新鲜。

3. **工作树限制**: 超过 50 个工作树时，`readBridgePointerAcrossWorktrees` 返回 null，限制当前目录。这在绝大多数场景下是安全的。

4. **模式严格性**: Zod 模式是严格的，任何额外字段或缺失字段都会导致解析失败并清理指针。

5. **并发安全**: 文件操作使用 `mkdir -p` 和原子写入，但不同进程/工作树的并发写入可能导致竞态。模块设计假设单一桥接实例每工作目录。

6. **扇出性能**: 工作树扇出并行读取使用 `Promise.all`，延迟约等于最慢的单个 stat。这对于通常的工作树数量（<10）是可接受的。
