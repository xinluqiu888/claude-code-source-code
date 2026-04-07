# App.tsx — 应用根组件，提供全局上下文

> **一句话总结**：为交互式会话提供顶层包装，注入 FPS 指标、统计数据和应用状态三个全局 Context。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/App.tsx` |
| 文件类型 | TSX |
| 代码行数 | 55 |
| 主要职责 | 作为整个应用的顶层容器，嵌套三层 Provider 向下传递全局上下文 |

---

## 功能概述

`App` 组件是 Claude Code 交互式会话的最顶层 React 包装组件。它不负责任何 UI 渲染，而是专注于建立全局上下文（Context）层级，让整个组件树都能访问到公共数据。

组件采用三层嵌套 Provider 结构：最外层是 `FpsMetricsProvider`（提供帧率指标），中间层是 `StatsProvider`（提供统计数据 Store），最内层是 `AppStateProvider`（提供应用核心状态）。

React Compiler（`react/compiler-runtime`）优化已被应用，通过记忆化（memoization）避免不必要的重新渲染。

---

## 核心内容详解

### 导入与依赖

| 模块 | 来源 |
|------|------|
| `FpsMetricsProvider` | `../context/fpsMetrics.js` |
| `StatsProvider`, `StatsStore` | `../context/stats.js` |
| `AppStateProvider`, `AppState` | `../state/AppState.js` |
| `onChangeAppState` | `../state/onChangeAppState.js` |
| `FpsMetrics` | `../utils/fpsTracker.js` |

### 主要类/函数/接口

**Props 类型**：

```ts
type Props = {
  getFpsMetrics: () => FpsMetrics | undefined;  // 获取 FPS 指标的函数
  stats?: StatsStore;                            // 可选的统计数据 Store
  initialState: AppState;                        // 应用初始状态
  children: React.ReactNode;                     // 子组件树
}
```

**App 函数组件**：接受 Props，返回三层嵌套的 Provider 树，最终将 `children` 渲染在最内层。

### 数据流与逻辑流程

```
App（Props）
  └─ FpsMetricsProvider（getFpsMetrics）
       └─ StatsProvider（stats store）
            └─ AppStateProvider（initialState, onChangeAppState）
                  └─ children
```

### 对外接口（导出内容）

- `export function App(props: Props): React.ReactNode`

---

## 设计要点

- **职责单一**：仅负责 Context 注入，不含任何业务逻辑或 UI 元素。
- **React Compiler 优化**：使用 `_c` 缓存机制，当 props 不变时跳过重新渲染。
- `onChangeAppState` 作为常量传入，不占用 memoization 槽位。

---

## 与其他文件的关系

- **依赖**：
  - `../context/fpsMetrics.js` — FPS 上下文
  - `../context/stats.js` — 统计数据上下文
  - `../state/AppState.js` — 应用状态
  - `../state/onChangeAppState.js` — 状态变更回调
- **被依赖**：应用入口文件（通常是 `main.tsx` 或 `index.tsx`）会挂载此组件。

---

## 注意事项

- `stats` prop 是可选的，如果不传则 `StatsProvider` 的 `store` 为 `undefined`。
- 此组件是 React Compiler 编译产物，手动修改时需注意不要破坏缓存逻辑（`$[N]` 槽位）。
