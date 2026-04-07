# modalContext.tsx — 模态框上下文管理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/context/modalContext.tsx`
- **类型**: React Context 组件
- **语言**: TypeScript + React

## 功能概述

由 FullscreenLayout 在渲染其 `modal` 槽位时设置 —— 这是用于斜杠命令对话框的绝对定位底部锚定面板。消费者使用此上下文来抑制顶级框架、调整 Select 分页大小，以及在标签切换时重置滚动。

## 核心内容详解

### 类型定义

```typescript
type ModalCtx = {
  rows: number
  columns: number
  scrollRef: RefObject<ScrollBoxHandle | null> | null
}
```

### 主要导出

1. **ModalContext** — React 上下文对象
   - `null` 表示不在模态槽位内部
   - 包含模态框的尺寸信息和滚动引用

2. **useIsInsideModal()** — 检查当前是否在模态框内部
   - 返回 boolean
   - 用于决定是否抑制顶级框架

3. **useModalOrTerminalSize(fallback)** — 获取模态框或终端尺寸
   - 在模态框内部时返回模态框尺寸
   - 否则返回提供的 fallback 尺寸
   - 用于限制可见内容高度

4. **useModalScrollRef()** — 获取模态框滚动引用
   - 返回 RefObject<ScrollBoxHandle | null> | null
   - 用于重置滚动位置

## 设计要点

1. **抑制顶级框架**:
   - `Pane` 组件使用 `useIsInsideModal()` 跳过全终端宽度的 `Divider`
   - FullscreenLayout 已经绘制了分隔线

2. **调整 Select 分页**:
   - 模态框内部区域比终端小 (rows - transcript peek - divider)
   - 使用 `useModalOrTerminalSize()` 替代 `useTerminalSize()`
   - 防止组件溢出可用行数

3. **标签切换时重置滚动**:
   - Tabs 组件通过 `selectedTabIndex` 为 ScrollBox 设置 key
   - 在标签切换时重新挂载，scrollTop 自动重置为 0
   - 无需 scrollTo() 时机控制

4. **React Compiler 优化**:
   - 使用 `_c` 函数进行记忆化
   - 缓存计算结果避免不必要的重新计算

## 与其他文件的关系

- **FullscreenLayout.tsx**: 在渲染 modal 槽位时提供 ModalContext
- **Pane.tsx**: 使用 `useIsInsideModal()` 决定是否渲染 Divider
- **Select 组件**: 使用 `useModalOrTerminalSize()` 限制可见选项数
- **Tabs.tsx**: 使用 scrollRef key 实现滚动重置
- **ScrollBox.tsx**: 通过 scrollRef 暴露滚动控制接口

## 注意事项

1. `null` 值表示不在模态槽位内部
2. 模态框的内部区域比完整终端尺寸小
3. scrollRef 可能为 null (不在模态框内)
4. 该上下文专门用于斜杠命令对话框场景
