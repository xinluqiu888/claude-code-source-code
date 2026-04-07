# flushGate.ts — 消息刷写状态门控

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/bridge/flushGate.ts`
- **文件类型**: TypeScript 模块
- **行数**: 约 72 行
- **主要职责**: 在初始刷写期间门控消息写入，防止新消息与历史消息交错

## 功能概述

`flushGate.ts` 实现了一个状态机，用于在桥接会话启动时的初始消息刷写阶段门控消息写入。当桥接会话开始时，历史消息通过单个 HTTP POST 发送到服务器。在此期间，新消息必须排队，防止它们到达服务器并与历史消息交错。

模块使用泛型类 `FlushGate<T>` 实现，可以存储任何类型的待处理项。生命周期包括：开始刷写（排队激活）、结束刷写（返回待处理项供排空）、丢弃（永久关闭传输）和停用（传输替换时不清除队列）。

该机制确保消息顺序的正确性，特别是在会话恢复或初始同步场景下，避免消息时间线混乱。

## 核心内容详解

### FlushGate 类

**泛型参数**: `T` - 待处理项的类型

**私有状态**:
```typescript
private _active = false      // 刷写是否进行中
private _pending: T[] = []   // 待处理项队列
```

### 属性

#### `active: boolean`（getter）
返回当前刷写状态。当为 `true` 时，`enqueue` 会将项加入队列；当为 `false` 时，`enqueue` 返回 `false`，调用者应直接发送。

#### `pendingCount: number`（getter）
返回当前队列中的待处理项数量，用于监控和调试。

### 方法

#### `start(): void`
标记刷写进行中。

**效果**:
- 设置 `_active = true`
- 后续 `enqueue` 调用开始排队项

**使用场景**: 在开始向服务器发送历史消息前调用。

#### `end(): T[]`
结束刷写并返回待处理项。

**效果**:
- 设置 `_active = false`
- 返回队列中的所有项（通过 `splice(0)` 提取并清空队列）
- 调用者负责发送返回的项

**使用场景**: 历史消息发送完成后调用，随后处理排队的新消息。

**示例**:
```typescript
gate.start()
// ... 发送历史消息 ...
const pending = gate.end()
for (const item of pending) {
  await send(item)  // 发送排队期间累积的消息
}
```

#### `enqueue(...items: T[]): boolean`
尝试将项加入队列。

**参数**: `...items` - 要加入的项（支持多个）

**返回**:
- `true` - 刷写进行中，项已加入队列
- `false` - 刷写未进行，调用者应直接发送

**行为**:
```typescript
if (!this._active) return false
this._pending.push(...items)
return true
```

**使用模式**:
```typescript
if (!gate.enqueue(message)) {
  // 刷写未进行，直接发送
  await transport.write(message)
}
```

#### `drop(): number`
丢弃所有待处理项（永久传输关闭）。

**效果**:
- 设置 `_active = false`
- 记录待处理项数量
- 清空队列（`this._pending.length = 0`）
- 返回丢弃的项数

**使用场景**: 传输永久关闭时调用，清理未发送的消息。

#### `deactivate(): void`
清除活跃标志但不丢弃队列。

**效果**:
- 设置 `_active = false`
- 保留 `_pending` 中的项

**使用场景**: 传输替换时（`onWorkReceived`）调用。新传输的刷写会排空待处理项。

## 设计要点

1. **泛型设计**: 使用泛型 `T` 使类可复用于不同消息类型（SDKMessage、内部消息等）。

2. **可变参数**: `enqueue` 接受剩余参数（`...items`），支持批量添加。

3. **状态机清晰**: 四个方法对应四个生命周期阶段，状态转换明确无歧义。

4. **所有权转移**: `end()` 通过 `splice(0)` 提取并清空队列，将待处理项所有权转移给调用者。

5. **停用 vs 丢弃**: 区分 `deactivate`（保留队列供新传输）和 `drop`（清理队列用于关闭），支持传输热切换。

6. **计数反馈**: `end()` 和 `drop()` 返回数量，便于调用者记录和监控。

7. **布尔返回值**: `enqueue` 返回布尔值而非抛出或静默处理，调用者必须显式处理分支。

## 与其他文件的关系

| 文件 | 关系描述 |
|------|----------|
| `replBridgeTransport.ts` | 可能使用 `FlushGate` 管理消息刷写 |
| `bridgeMain.ts` / `replBridge.ts` | 在初始同步和传输替换时管理门控生命周期 |
| `inboundMessages.ts` | 可能作为消息处理的协调点 |

## 注意事项

1. **内存增长**: 如果 `start()` 后长时间不调用 `end()`，队列可能无限增长。调用者应确保刷写有超时机制。

2. **线程安全**: 类未使用锁或原子操作，设计用于单线程 JavaScript 事件循环。并发调用可能导致竞态。

3. **泛型约束**: 虽然 `T` 无显式约束，但实际使用通常期望 `T` 可复制（非包含循环引用）。

4. **批量语义**: `enqueue(...items)` 原子性添加所有项，要么全部入队，要么全部不入（如果未激活）。

5. **重复 end**: 多次调用 `end()` 是安全的（返回空数组），但可能表明逻辑错误。

6. **混合模式**: 不支持部分刷写——一旦 `end()`，所有待处理项被提取，不能保留部分。

7. **deactivate 后 enqueue**: `deactivate()` 后调用 `enqueue` 返回 `false`，项不会被加入队列。

8. **drop 与 deactivate 选择**:
   - 使用 `drop` 当传输永久关闭（如进程退出）
   - 使用 `deactivate` 当传输替换（如重连后新 WebSocket）
