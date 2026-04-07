# overlayContext.tsx — 覆盖层状态跟踪与 Escape 键协调

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/context/overlayContext.tsx`
- **类型**: React Hook 集合
- **语言**: TypeScript + React

## 功能概述

该文件解决了当覆盖层 (如带有 onCancel 的 Select) 打开时 Escape 键的处理问题。CancelRequestHandler 需要知道覆盖层何时处于活动状态，以便在用户只想关闭覆盖层时不取消请求。

## 核心内容详解

### 主要导出

1. **useRegisterOverlay(id, enabled?)** — 自动将组件注册为活动覆盖层
   - 在挂载时注册，在卸载时注销
   - 支持条件注册 (通过 enabled 参数)
   - 强制重新渲染以进行完整差异计算 (避免 blit 快速路径复制过时的覆盖层单元格)

2. **useIsOverlayActive()** — 检查是否有任何覆盖层当前处于活动状态
   - 响应式 —— 当覆盖层状态变化时组件将重新渲染

3. **useIsModalOverlayActive()** — 检查是否有模态覆盖层处于活动状态
   - 区分模态覆盖层 (如 Select 对话框) 和非模态覆盖层 (如 autocomplete)
   - 非模态覆盖层不会禁用 TextInput 焦点

### 常量定义

```typescript
// 不禁用 TextInput 焦点的非模态覆盖层
const NON_MODAL_OVERLAYS = new Set(['autocomplete'])
```

## 设计要点

1. **自动注册/注销**: 
   - 使用 useEffect 在挂载时注册覆盖层
   - 返回清理函数在卸载时注销覆盖层
   - 无需手动清理或状态管理

2. **覆盖层关闭时的强制重新渲染**:
   - 使用 useLayoutEffect 在覆盖层关闭时强制下一帧进行完整差异计算
   - 防止 tall 覆盖层 (如带有 20 行预览的 FuzzyPicker) 卸载时留下幽灵标题/分隔符

3. **条件注册支持**:
   - 允许基于组件属性条件注册
   - 例如：仅在提供了 onCancel 时才注册

4. **直接使用 Context**:
   - 直接使用 useContext(AppStoreContext) 以便在 AppStateProvider 外部渲染时成为空操作
   - 适用于不需要完整应用状态树的隔离组件测试

## 与其他文件的关系

- **CancelRequestHandler**: 使用 `useIsOverlayActive()` 检查是否可以取消请求
- **AppState.tsx**: 使用 AppStoreContext 进行状态管理
- **Select 组件**: 使用 `useRegisterOverlay('select', !!onCancel)` 注册自身

## 注意事项

1. 覆盖层 ID 是唯一的标识符，如 'select'、'multi-select'、'autocomplete'
2. useLayoutEffect 确保清理在微任务延迟的 onRender 之前同步运行
3. 该机制可防止高覆盖层在卸载时将过时的单元格复制到不再可达的行中
4. 模态覆盖层用于决定 TextInput 焦点行为
