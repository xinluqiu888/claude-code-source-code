# autoDream.ts — 自动梦境（后台记忆整合）

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/autoDream/autoDream.ts`
- **作用域**: 后台记忆整合，自动触发 /dream 提示
- **主要导出**:
  - `initAutoDream`: 初始化自动梦境
  - `executeAutoDream`: 执行自动梦境（每回合入口点）

## 功能概述

在特定条件满足时，以 forked 子代理形式自动执行 `/dream` 提示。通过时间门、会话门和锁机制控制触发，实现后台记忆整合而不干扰用户工作。

## 核心内容详解

### 触发门控（按成本排序）

```
1. 时间门: hours since lastConsolidatedAt >= minHours (一次 stat)
2. 会话门: mtime > lastConsolidatedAt 的会话数 >= minSessions
3. 锁: 无其他进程正在进行整合
```

### 配置类型

```typescript
type AutoDreamConfig = {
  minHours: number      // 最小间隔小时数（默认24）
  minSessions: number   // 最小会话数（默认5）
}
```

### 状态管理

状态保存在 `initAutoDream()` 的闭包内（非模块级），支持测试隔离：
```typescript
let runner: ((context, appendSystemMessage?) => Promise<void>) | null = null
let lastSessionScanAt = 0  // 上次会话扫描时间
```

### 核心函数

#### `initAutoDream()`
初始化自动梦境系统：

**调用时机**:
- 启动时（与 `initExtractMemories` 一起从 `backgroundHousekeeping` 调用）
- 测试的 `beforeEach`（获得新鲜闭包）

**内部 runner 逻辑**:

**1. 强制模式检查** (`isForced()`):
- Ant-only 测试覆盖
- 绕过启用/时间/会话门控，但不绕过锁

**2. 门控检查** (`isGateOpen()`):
```typescript
- Kairos 模式活动 → false
- 远程模式 → false
- 自动记忆未启用 → false
- 自动梦境未启用 → false
```

**3. 时间门**:
```typescript
hoursSince = (Date.now() - lastAt) / 3_600_000
if (hoursSince < minHours) return
```

**4. 扫描节流** (`SESSION_SCAN_INTERVAL_MS = 10分钟`):
防止时间门通过但会话门不通过时的频繁扫描。

**5. 会话门**:
- 获取自上次整合以来访问的会话
- 排除当前会话（其 mtime 总是最近的）
- 检查数量是否达到 `minSessions`

**6. 锁获取**:
- 非强制模式：调用 `tryAcquireConsolidationLock()`
- 强制模式：跳过获取，使用现有 mtime

**7. 执行整合**:
```typescript
// 注册任务
const taskId = registerDreamTask(setAppState, {
  sessionsReviewing: sessionIds.length,
  priorMtime,
  abortController,
})

// 构建提示
const prompt = buildConsolidationPrompt(memoryRoot, transcriptDir, extra)

// Fork 执行
const result = await runForkedAgent({
  promptMessages: [createUserMessage({ content: prompt })],
  cacheSafeParams: createCacheSafeParams(context),
  canUseTool: createAutoMemCanUseTool(memoryRoot),
  querySource: 'auto_dream',
  forkLabel: 'auto_dream',
  skipTranscript: true,
  overrides: { abortController },
  onMessage: makeDreamProgressWatcher(taskId, setAppState),
})
```

**8. 完成处理**:
- 标记任务完成 (`completeDreamTask`)
- 可选追加系统消息（显示 "Improved N memories"）
- 记录遥测事件 (`tengu_auto_dream_completed`)

**9. 错误处理**:
- 用户取消：检测 `abortController.signal.aborted`
- 其他错误：记录失败，回滚锁，标记任务失败

#### `executeAutoDream(context, appendSystemMessage?)`
每回合入口点：

**成本**（启用时）：一次 GB 缓存读取 + 一次 stat

**参数**:
- `context`: REPLHookContext
- `appendSystemMessage`: 可选的系统消息追加函数

### 进度监视器

#### `makeDreamProgressWatcher(taskId, setAppState)`
监视 forked 代理的消息：

**处理**:
- 提取文本块（推理/摘要）
- 统计 tool_use 块数量
- 收集 FileEdit/FileWrite 工具的文件路径

**调用**: `addDreamTurn(taskId, { text, toolUseCount }, touchedPaths, setAppState)`

### 遥测事件

| 事件 | 数据 |
|------|------|
| `tengu_auto_dream_fired` | hours_since, sessions_since |
| `tengu_auto_dream_completed` | cache_read, cache_created, output, sessions_reviewed |
| `tengu_auto_dream_failed` | - |

### 工具约束

自动梦境中 Bash 工具限制为只读命令：
```
ls, find, grep, cat, stat, wc, head, tail 等
```

写入、重定向或修改状态的操作被拒绝。

## 设计要点

1. **后台执行**: 使用 `runForkedAgent` 在后台运行，不阻塞主循环
2. **多级门控**: 按成本排序的门控，减少不必要的检查
3. **锁机制**: 文件锁防止多进程冲突
4. **进度追踪**: DreamTask 系统显示进度和取消按钮
5. **缓存安全**: 使用 `createCacheSafeParams` 处理缓存
6. **节流机制**: 10分钟扫描节流防止频繁检查
7. **回滚支持**: 失败时回滚锁时间戳，允许重试

## 与其他文件的关系

- **config.ts**: 提供 `isAutoDreamEnabled`
- **consolidationPrompt.ts**: 提供 `buildConsolidationPrompt`
- **consolidationLock.ts**: 提供锁管理函数
- **utils/forkedAgent.ts**: 提供 `runForkedAgent`, `createCacheSafeParams`
- **utils/messages.ts**: 提供 `createUserMessage`, `createMemorySavedMessage`
- **tasks/DreamTask/DreamTask.ts**: 提供任务管理函数
- **extractMemories/extractMemories.ts**: 提供 `createAutoMemCanUseTool`

## 注意事项

1. **Kairos 模式**: 使用磁盘技能梦境，不触发自动梦境
2. **远程模式**: 禁用自动梦境
3. **跳过转录**: `skipTranscript: true`，梦境过程不记录在主转录中
4. **用户取消**: 支持通过背景任务对话框取消
5. **测试隔离**: 状态在闭包内，支持 `beforeEach` 重置
