# promptOverlayContext.tsx — 浮动覆盖层内容传送门

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/context/promptOverlayContext.tsx`
- **类型**: React Context 组件
- **语言**: TypeScript + React

## 功能概述

该文件提供了一种 Portal 机制，用于在提示符(prompt)上方渲染浮动内容，使其能够逃离 `FullscreenLayout` 底部槽位的 `overflowY:hidden` 裁剪限制。这是解决 Ink 组件库裁剪堆栈问题的关键技术方案。

## 核心内容详解

### 主要导出

1. **PromptOverlayProvider** — 提供浮动覆盖层的状态管理
2. **usePromptOverlay** — 获取建议数据 (suggestions, selectedSuggestion, maxColumnWidth)
3. **usePromptOverlayDialog** — 获取任意对话框节点 (如 AutoModeOptInDialog)
4. **useSetPromptOverlay** — 注册建议数据，卸载时自动清除
5. **useSetPromptOverlayDialog** — 注册对话框节点，卸载时自动清除

### 数据/设置器上下文分离设计

文件采用数据/设置器上下文对分离的策略：
- **DataContext**: 存储建议数据 (PromptOverlayData)
- **SetContext**: 存储数据设置函数
- **DialogContext**: 存储对话框节点 (ReactNode)
- **SetDialogContext**: 存储对话框设置函数

这种分离确保写入者不会因自己的写入而重新渲染 —— 设置器上下文是稳定的。

### 类型定义

```typescript
export type PromptOverlayData = {
  suggestions: SuggestionItem[]
  selectedSuggestion: number
  maxColumnWidth?: number
}
```

## 设计要点

1. **双通道设计**: 
   - `useSetPromptOverlay` — 用于斜杠命令建议数据 (由 PromptInputFooter 写入)
   - `useSetPromptOverlayDialog` — 用于任意对话框节点 (如 AutoModeOptInDialog，由 PromptInput 写入)

2. **自动清理机制**: 使用 useEffect 的返回函数在组件卸载时自动清除覆盖层数据

3. **非全屏渲染兼容**: 在非全屏渲染模式下，这些 hook 是空操作 (no-op)，内容将以内联方式渲染

4. **React Compiler 优化**: 使用 `_c` 函数进行编译器优化，带有记忆化缓存

## 与其他文件的关系

- **FullscreenLayout.tsx**: 读取两个通道的数据并在裁剪槽位外部渲染它们
- **PromptInputFooter.tsx**: 使用 `useSetPromptOverlay` 注册建议数据
- **PromptInput.tsx**: 使用 `useSetPromptOverlayDialog` 注册对话框节点

## 注意事项

1. 浮动覆盖层使用 `position:absolute bottom="100%"` 浮动在提示符上方
2. Ink 的裁剪堆栈会相交所有后代元素，因此内容需要被渲染在裁剪槽位外部
3. 该机制是负载承载的 (load-bearing)，用于防止高粘贴内容挤压 ScrollBox
4. 在 Provider 外部调用设置 hook 是安全的，不会产生副作用
