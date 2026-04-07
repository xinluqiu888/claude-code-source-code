# compactWarningHook.ts — 压缩警告 React Hook

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/compact/compactWarningHook.ts`
- **作用域**: React Hook 层
- **主要导出**:
  - `useCompactWarningSuppression`: 订阅压缩警告抑制状态

## 功能概述

提供 React Hook 用于订阅压缩警告抑制状态。该文件单独存在以保持 `compactWarningState.ts` 的 React-free 特性。

## 核心内容详解

```typescript
import { useSyncExternalStore } from 'react'
import { compactWarningStore } from './compactWarningState.js'

export function useCompactWarningSuppression(): boolean {
  return useSyncExternalStore(
    compactWarningStore.subscribe,
    compactWarningStore.getState,
  )
}
```

## 设计要点

1. **关注点分离**: React 相关代码与纯状态管理分离
2. **useSyncExternalStore**: 使用 React 18 推荐的外部状态订阅方式
3. **类型安全**: 返回布尔值表示是否抑制警告

## 与其他文件的关系

- **compactWarningState.ts**: 提供状态存储实例
- **UI 组件**: 使用此 hook 获取警告显示状态

## 注意事项

- `microCompact.ts` 需要导入 `compactWarningState.ts` 而不引入 React，因此需要分离
