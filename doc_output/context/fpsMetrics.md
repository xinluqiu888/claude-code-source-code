# fpsMetrics.tsx — FPS 性能指标上下文

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/context/fpsMetrics.tsx`
- **类型**: React Context 组件
- **语言**: TypeScript + React

## 功能概述

提供 FPS (每秒帧数) 性能指标的 React 上下文，用于在应用中监控和访问渲染性能数据。

## 核心内容详解

### 类型定义

```typescript
type FpsMetricsGetter = () => FpsMetrics | undefined
```

### 主要导出

1. **FpsMetricsProvider** — FPS 指标提供者
   - 接收 `getFpsMetrics` 函数和子组件
   - 通过 Context 提供 FPS 指标访问

2. **useFpsMetrics()** — FPS 指标 hook
   - 返回 FpsMetricsGetter 函数
   - 调用该函数可获取当前 FPS 指标

### FpsMetrics 类型

从 `../utils/fpsTracker.js` 导入，通常包含：
- 当前 FPS 值
- 平均 FPS
- 帧时间统计
- 性能趋势数据

## 设计要点

1. **函数式访问**:
   - 提供 getter 函数而非直接提供值
   - 允许消费者按需获取最新指标
   - 避免不必要的重新渲染

2. **可选值**:
   - 返回值可能为 undefined
   - 消费者需要处理未定义情况

3. **React Compiler 优化**:
   - 使用 `_c` 函数进行编译器优化
   - 缓存 provider 渲染结果

4. **轻量级设计**:
   - 仅传递 getter 函数，不传递复杂状态
   - 最小化对渲染性能的影响

## 与其他文件的关系

- **../utils/fpsTracker.js**: FpsMetrics 类型定义和 FPS 跟踪逻辑
- **性能监控组件**: 使用 useFpsMetrics 获取并显示性能数据

## 注意事项

1. FPS 跟踪需要在应用的其他地方初始化
2. Provider 仅提供访问机制，不负责实际跟踪
3. 适用于开发和调试阶段的性能监控
4. 生产环境可能需要条件编译或懒加载
