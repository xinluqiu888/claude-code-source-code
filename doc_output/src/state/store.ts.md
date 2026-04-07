# store.ts — 通用状态存储实现

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/state/store.ts`
- **类型**: TypeScript 模块
- **导出内容**: `Store<T>` 类型、`createStore<T>()` 函数
- **依赖关系**: 无外部依赖

## 功能概述

本文件提供了一个极简的、基于订阅模式的状态管理 Store 实现。这是一个通用、类型安全的状态容器，为应用程序提供类似 Redux 但更轻量的状态管理能力。它是 Claude Code 状态管理的基础构建块。

## 核心内容详解

### 1. 类型定义 (第1-8行)

```typescript
type Listener = () => void
type OnChange<T> = (args: { newState: T; oldState: T }) => void

export type Store<T> = {
  getState: () => T
  setState: (updater: (prev: T) => T) => void
  subscribe: (listener: Listener) => () => void
}
```

**Store 接口**:
- `getState()`: 获取当前状态
- `setState(updater)`: 通过更新函数修改状态
- `subscribe(listener)`: 订阅状态变化，返回取消订阅函数

### 2. createStore() 函数 (第10-34行)

创建新的 Store 实例。

**参数**:
- `initialState: T`: 初始状态值
- `onChange?: OnChange<T>`: 可选的状态变化回调

**实现细节**:

```typescript
export function createStore<T>(
  initialState: T,
  onChange?: OnChange<T>,
): Store<T> {
  let state = initialState
  const listeners = new Set<Listener>()

  return {
    getState: () => state,

    setState: (updater: (prev: T) => T) => {
      const prev = state
      const next = updater(prev)
      if (Object.is(next, prev)) return  // 引用相等时跳过
      state = next
      onChange?.({ newState: next, oldState: prev })
      for (const listener of listeners) listener()
    },

    subscribe: (listener: Listener) => {
      listeners.add(listener)
      return () => listeners.delete(listener)
    },
  }
}
```

**关键特性**:
1. **不可变性**: 使用更新函数模式 `(prev: T) => T` 确保状态不可变
2. **引用优化**: 使用 `Object.is()` 检查状态是否真正改变，避免不必要的更新
3. **同步通知**: 状态变化时同步通知所有监听器
4. **内存管理**: 订阅返回取消订阅函数，防止内存泄漏

## 设计要点

1. **极简设计**: 只有 34 行代码，无外部依赖
2. **类型安全**: 完整的 TypeScript 泛型支持
3. **性能优化**: 引用相等检查避免不必要的渲染
4. **可扩展**: 通过 `onChange` 回调可以添加中间件功能
5. **React 友好**: 订阅模式天然适合 React 的 useSyncExternalStore

## 使用示例

```typescript
// 创建 store
const store = createStore({ count: 0 }, ({ newState, oldState }) => {
  console.log('State changed:', oldState, '->', newState)
})

// 订阅变化
const unsubscribe = store.subscribe(() => {
  console.log('Listener called, new state:', store.getState())
})

// 更新状态
store.setState(prev => ({ count: prev.count + 1 }))

// 取消订阅
unsubscribe()
```

## 与其他文件的关系

- **AppStateStore.ts**: 使用此 Store 作为 AppState 的基础
- **onChangeAppState.ts**: 通过 `onChange` 回调监听 AppState 变化
- **React 组件**: 通过订阅机制与 React 组件集成

## 注意事项

1. **不要直接修改状态**: 总是通过 `setState` 更新
2. **更新函数应纯**: 不要在更新函数中产生副作用
3. **监听器同步执行**: 所有监听器同步执行，避免长时间操作
4. **深度比较**: 使用 `Object.is()` 进行浅比较，嵌套对象变化需要返回新引用

## 与 Redux 的对比

| 特性 | Redux | 本 Store |
|------|-------|----------|
| Actions | 需要定义 | 使用更新函数直接更新 |
| Reducers | 需要编写 | 内联在 setState 中 |
| Middleware | 支持丰富 | 通过 onChange 简单支持 |
| DevTools | 有专用工具 | 需要自行实现 |
| 代码量 | 较多 | 34 行 |
| 学习曲线 | 较陡 | 极简 |
