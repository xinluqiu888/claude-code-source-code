# capacityWake.ts — 桥接轮询循环的容量唤醒原语

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/bridge/capacityWake.ts`
- **文件类型**: TypeScript 模块
- **行数**: 约 57 行
- **主要职责**: 提供桥接轮询循环的容量唤醒机制，支持在容量受限时休眠并在条件满足时提前唤醒

## 功能概述

`capacityWake.ts` 封装了桥接轮询循环的容量唤醒原语。当桥接达到容量上限（同时处理的最大会话数）时，需要进入休眠状态等待容量释放。该模块提供了统一的机制来处理两种唤醒信号：外部循环的中断信号（如关机）和容量释放信号（会话完成或传输丢失）。

此前 `replBridge.ts` 和 `bridgeMain.ts` 中的轮询循环各自实现了几乎相同的字节级代码。本模块将其提取为可复用的组件，供两种桥接核心共享使用。

核心机制使用 `AbortController` 来生成可中断的 `AbortSignal`，并通过合并两个信号（外部信号和容量唤醒信号）来实现"任一信号触发即唤醒"的语义。

## 核心内容详解

### 类型定义

#### `CapacitySignal` 类型
```typescript
{
  signal: AbortSignal;   // 合并后的信号
  cleanup: () => void;   // 清理函数（移除监听器）
}
```

#### `CapacityWake` 类型
```typescript
{
  signal(): CapacitySignal;  // 创建合并信号
  wake(): void;              // 触发容量唤醒
}
```

### createCapacityWake 函数

**签名**: `createCapacityWake(outerSignal: AbortSignal): CapacityWake`

**参数**: `outerSignal` - 外部循环的中断信号（如关机信号）

**实现机制**:
1. 创建内部 `wakeController` AbortController
2. `signal()` 方法合并两个信号：
   - 如果任一信号已中止，立即返回已中止的合并信号
   - 否则在两个信号上添加 `abort` 事件监听器
   - 任一信号触发时，合并信号也会触发
   - 返回包含清理函数的 `CapacitySignal`
3. `wake()` 方法：
   - 中止当前 `wakeController`
   - 创建新的 `wakeController` 以供下次使用

**信号合并逻辑**:
```typescript
if (outerSignal.aborted || wakeController.signal.aborted) {
  merged.abort()  // 任一已中止则立即中止
}
// 否则在两个信号上添加监听器
outerSignal.addEventListener('abort', abort, { once: true })
wakeController.signal.addEventListener('abort', abort, { once: true })
```

### 使用模式

典型的轮询循环使用模式：

```typescript
const capacityWake = createCapacityWake(outerSignal)

while (!outerSignal.aborted) {
  if (atCapacity) {
    // 创建合并信号
    const { signal, cleanup } = capacityWake.signal()
    
    // 等待信号触发或超时
    await sleep(someDelay, signal)
    
    // 正常完成时清理监听器
    cleanup()
    
    // 继续循环检查条件
    continue
  }
  
  // 正常处理...
}
```

当容量释放时（会话完成、传输丢失），调用 `capacityWake.wake()` 提前唤醒休眠的轮询循环。

## 设计要点

1. **双信号合并**: 将外部中断信号和容量唤醒信号合并为单一信号，简化调用者的等待逻辑。

2. **清理函数**: 返回清理函数供正常完成时调用，避免内存泄漏。使用 `{ once: true }` 进一步确保监听器自动清理。

3. **控制器重建**: `wake()` 中止当前控制器后立即重建新控制器，确保下次 `signal()` 调用获得新的、未中止的信号。

4. **即时检查**: `signal()` 首先检查任一信号是否已中止，如果是则立即返回已中止信号，避免不必要的监听器注册。

5. **无副作用**: 函数是纯函数，不修改外部状态，唯一的副作用是内部控制器的创建和中止。

6. **竞态安全**: 使用 `AbortController` 的 DOM 标准语义，确保信号状态一致性。

## 与其他文件的关系

| 文件 | 关系描述 |
|------|----------|
| `replBridge.ts` | 使用 `createCapacityWake` 实现轮询循环的容量管理 |
| `bridgeMain.ts` | 使用 `createCapacityWake` 实现轮询循环的容量管理 |

## 注意事项

1. **清理函数必须调用**: 虽然使用了 `{ once: true }`，但在正常完成时调用 `cleanup()` 是良好实践，确保立即释放资源而非等待 GC。

2. **重建时机**: `wake()` 立即重建控制器，这是有意的设计——确保即使 `wake()` 被多次调用，后续 `signal()` 调用仍能获得有效信号。

3. **AbortSignal 兼容性**: 依赖标准的 `AbortSignal`/`AbortController` API，需要 Node.js 15+ 或 polyfill。

4. **无超时管理**: 模块本身不处理超时，调用者需要将合并信号传递给支持 AbortSignal 的睡眠函数（如 `setTimeout` 包装器）。

5. **线程安全**: `AbortController` 是单线程的，本模块设计用于单线程的 JavaScript 事件循环环境。

6. **内存泄漏防护**: 如果信号触发（中止），`{ once: true }` 自动清理监听器；如果正常完成，调用者负责调用 `cleanup()`。
