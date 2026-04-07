# review/ultrareviewCommand.tsx

## 文件描述
Ultrareview 命令组件 - 带超额对话框的代码审查界面

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | React组件 |
| 导出 | call函数 |
| 功能 | 提供代码审查功能，处理超额计费 |
| 组件 | OverageDialog, ReviewUI |

## 函数概述

### call
主组件函数，处理代码审查流程：
1. 显示超额计费对话框（如需要）
2. 执行代码审查
3. 显示审查结果

### 函数签名
```typescript
export async function call(
  onDone: LocalJSXCommandOnDone,
  context: LocalJSXCommandContext,
): Promise<React.ReactNode | null>
```

## 核心内容

### 组件渲染流程
```typescript
// 1. 显示超额对话框（如果需要）
<OverageDialog
  onConfirm={handleConfirm}
  onCancel={handleCancel}
/>

// 2. 显示审查界面
<ReviewUI
  changes={changes}
  onComplete={onDone}
/>
```

### 超额处理
- 检查预估费用是否超过用户配额
- 显示超额确认对话框
- 用户确认后继续审查

### 审查流程
- 获取代码变更
- 分析代码质量
- 生成审查报告
- 显示审查结果

## 设计点

1. **计费透明**：明确显示预估费用和超额情况
2. **用户确认**：超额时要求用户明确确认
3. **组件化**：使用独立的 OvergaDialog 和 ReviewUI 组件
4. **异步流程**：完整的异步处理链

## 与其他文件的关系

- 导入 `OverageDialog` 组件
- 导入 `ReviewUI` 组件
- 被 `reviewRemote.ts` 调用
- 使用 `LocalJSXCommandContext` 类型

## 注意事项

- 超额对话框仅在需要时显示
- 用户取消时需优雅处理
- 审查过程可能较长，需要loading状态
- 错误处理需完善
