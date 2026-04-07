# useBackgroundTaskNavigation.ts — 后台任务键盘导航

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/useBackgroundTaskNavigation.ts`
- **类型**: React Hook
- **导出函数**: `useBackgroundTaskNavigation`
- **依赖**: React useEffect/useRef, ink useInput, AppState

## 功能概述

本 Hook 提供后台任务（teammates/swarm 成员）的键盘导航功能：
1. Shift+Up/Down 在 Leader 和 Teammates 之间切换
2. 支持查看 Teammate 转录本
3. 支持终止运行中的 Teammate 任务
4. 处理背景任务对话框的打开

## 核心内容详解

### stepTeammateSelection 函数

选择索引规则：
- `-1`: Leader（默认）
- `0..n-1`: Teammates（按字母排序）
- `n`: "Hide" 行（隐藏转录树）

**步进逻辑**
```typescript
const next = delta === 1
  ? cur >= maxIdx ? -1 : cur + 1      // 向下：循环到开头
  : cur <= -1 ? maxIdx : cur - 1      // 向上：循环到结尾
```

### 键盘处理

| 按键 | 模式 | 行为 |
|------|------|------|
| Shift+Up/Down | 任意 | 在 Teammates 间切换或打开后台任务对话框 |
| Escape | viewing-agent | 运行中：中止当前工作；已完成：退出视图 |
| Escape | selecting-agent | 退出选择模式 |
| 'f' | selecting-agent | 查看选中 Teammate 的转录本 |
| 'k' | selecting-agent | 终止选中的 Teammate |
| Enter | selecting-agent | -1: 返回 Leader；n: 隐藏树；其他: 查看转录 |

### 状态管理

**Teammate 数量变化处理**
```typescript
useEffect(() => {
  // Teammate 从有到无时重置选择
  // 边界检查时钳制索引值
}, [teammateCount])
```

## 设计要点

### 1. 视图模式区分

- `viewSelectionMode`: 'none' | 'selecting-agent' | 'viewing-agent'
- 不同模式下相同按键有不同行为

### 2. 中止 vs 终止

- **中止 (Abort)**: 停止当前对话轮次，Teammate 保持存活
- **终止 (Kill)**: 完全停止 Teammate 任务

### 3. 索引管理

- 使用 ref 跟踪上一次 Teammate 数量
- 数量变化时自动调整选择索引

## 与其他文件的关系

- **teammateViewHelpers.ts**: 提供 `enterTeammateView` 和 `exitTeammateView`
- **InProcessTeammateTask.ts**: 提供 `getRunningTeammatesSorted` 和 `kill`
- **AppState**: 管理选择状态和任务状态

## 注意事项

1. **向后兼容**: 使用 useInput 作为临时桥接，直到 REPL 迁移到 onKeyDown
2. **空输入保护**: 无 Teammate 时 Shift+Up/Down 打开后台任务对话框
3. **循环导航**: 选择索引在 Leader-Teammates-Hide 之间循环
