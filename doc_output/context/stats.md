# stats.tsx — 统计指标收集与管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/context/stats.tsx`
- **类型**: React Context 组件 + 统计存储实现
- **语言**: TypeScript + React

## 功能概述

提供应用性能和行为统计的收集、存储和管理系统。支持计数器、仪表盘、计时器、直方图和集合等多种统计类型，并在进程退出时自动刷新指标到项目配置。

## 核心内容详解

### StatsStore 接口

```typescript
export type StatsStore = {
  increment(name: string, value?: number): void  // 递增计数器
  set(name: string, value: number): void         // 设置仪表盘值
  observe(name: string, value: number): void     // 观察直方图值
  add(name: string, value: string): void         // 添加到集合
  getAll(): Record<string, number>               // 获取所有指标
}
```

### 统计类型

1. **计数器 (Counter)**: 单调递增的数值
2. **仪表盘 (Gauge)**: 任意数值
3. **计时器/直方图 (Timer/Histogram)**: 
   - 支持百分位数计算 (p50, p95, p99)
   - 使用水库抽样算法 (Algorithm R)
   - 最大样本数: 1024 (RESERVOIR_SIZE)
4. **集合 (Set)**: 唯一字符串值的集合

### 主要导出

1. **createStatsStore()** — 创建统计存储实例
2. **StatsProvider** — 统计上下文提供者
3. **useStats()** — 获取 StatsStore 实例
4. **useCounter(name)** — 返回计数器递增函数
5. **useGauge(name)** — 返回仪表盘设置函数
6. **useTimer(name)** — 返回计时器观察函数
7. **useSet(name)** — 返回集合添加函数

### 百分位数计算

```typescript
function percentile(sorted: number[], p: number): number
```
- 使用线性插值计算百分位数
- 支持小数索引的插值计算

## 设计要点

1. **水库抽样 (Reservoir Sampling)**:
   - 当样本数超过 1024 时使用算法 R
   - 保持均匀随机抽样，不依赖数据流长度
   - 内存使用有界

2. **直方图统计**:
   - 计算 count、min、max、avg
   - 计算 p50、p95、p99 百分位数
   - 使用排序后的水库样本

3. **进程退出刷新**:
   - 监听 `process.on('exit')` 事件
   - 将指标保存到项目配置的 `lastSessionMetrics`
   - 确保数据不丢失

4. **外部存储支持**:
   - StatsProvider 接受可选的 `store` 属性
   - 支持注入外部统计存储
   - 否则使用内部创建的存储

5. **记忆化回调**:
   - useCounter、useGauge、useTimer、useSet 使用 useCallback
   - 仅在 name 或 store 变化时重新创建

## 与其他文件的关系

- **../utils/config.js**: saveCurrentProjectConfig 用于持久化指标
- **性能监控**: 使用各种统计 hook 收集性能数据

## 注意事项

1. 直方图使用有界内存 (1024 样本)
2. 百分位数计算需要排序，有一定性能开销
3. 进程退出时自动刷新，但非正常退出可能丢失数据
4. 集合统计输出为集合大小，而非集合内容
5. 支持外部存储注入用于测试或特殊场景
