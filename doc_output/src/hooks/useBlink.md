# useBlink.ts — 同步闪烁动画 Hook

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/useBlink.ts`
- **类型**: React Hook
- **导出函数**: `useBlink`
- **依赖**: ink useAnimationFrame, useTerminalFocus

## 功能概述

本 Hook 提供同步的闪烁动画效果：
1. 所有实例共享同一时间基准，保持同步闪烁
2. 元素不可见时自动暂停动画
3. 终端失焦时暂停动画
4. 可自定义闪烁间隔

## 核心内容详解

### 参数

```typescript
function useBlink(
  enabled: boolean,                    // 是否启用闪烁
  intervalMs: number = 600            // 闪烁间隔（毫秒）
): [ref: (element: DOMElement | null) => void, isVisible: boolean]
```

### 实现原理

**时间驱动状态**
```typescript
const [ref, time] = useAnimationFrame(enabled && focused ? intervalMs : null)
const isVisible = Math.floor(time / intervalMs) % 2 === 0
```

- `useAnimationFrame` 返回递增的时间戳
- 将时间转换为闪烁状态（可见/隐藏）
- 所有实例使用同一时间源，自然同步

### 状态表

| enabled | focused | 返回值 |
|---------|---------|--------|
| false | any | `[ref, true]` |
| true | false | `[ref, true]` |
| true | true | `[ref, isVisible]` |

## 设计要点

### 1. 共享时间源

所有 `useBlink` 实例订阅同一个动画帧源：
- 保证所有闪烁元素完全同步
- 减少多个定时器的性能开销

### 2. 自动暂停

- 元素不可见（滚出屏幕）：`useAnimationFrame` 自动暂停
- 终端失焦：`useTerminalFocus()` 返回 false，暂停更新

### 3. 资源优化

`useAnimationFrame(null)` 停止动画循环，节省 CPU

## 与其他文件的关系

- **ink**: 提供 `useAnimationFrame` 和 `useTerminalFocus`
- **使用示例**: 加载指示器、光标闪烁、状态指示点

## 注意事项

1. **初始状态**: 禁用时返回 `true`（可见），避免闪烁开始前的隐藏
2. **精度依赖**: 动画帧率依赖终端刷新率
3. **时间漂移**: 长时间运行可能有轻微漂移，但不影响视觉效果
4. **间隔限制**: 最小间隔受限于动画帧率（通常 60fps = ~16ms）
