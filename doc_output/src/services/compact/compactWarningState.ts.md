# compactWarningState.ts — 压缩警告状态管理

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/compact/compactWarningState.ts`
- **作用域**: 压缩警告抑制状态管理
- **主要导出**:
  - `compactWarningStore`: 状态存储实例
  - `suppressCompactWarning()`: 抑制压缩警告
  - `clearCompactWarningSuppression()`: 清除警告抑制

## 功能概述

管理"上下文剩余直到自动压缩"警告的显示状态。在成功压缩后抑制警告，因为需要等待下一次 API 响应才能获得准确的 token 计数。

## 核心内容详解

```typescript
// 创建状态存储
export const compactWarningStore = createStore<boolean>(false)

// 抑制警告（压缩成功后调用）
export function suppressCompactWarning(): void {
  compactWarningStore.setState(() => true)
}

// 清除抑制（新的压缩尝试开始时调用）
export function clearCompactWarningSuppression(): void {
  compactWarningStore.setState(() => false)
}
```

## 设计要点

1. **状态隔离**: 使用独立的 store 避免与 React 耦合
2. **简单布尔状态**: true 表示抑制警告，false 表示正常显示
3. **生命周期管理**: 
   - 压缩成功 -> 抑制警告
   - 新压缩尝试 -> 清除抑制

## 与其他文件的关系

- **compactWarningHook.ts**: React hook 订阅此状态
- **microCompact.ts**: 调用 `suppressCompactWarning` 和 `clearCompactWarningSuppression`

## 注意事项

- 保持文件 React-free，以便 microCompact.ts 可以在非 React 环境中导入
