# diff.tsx — 差异查看界面

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/diff/diff.tsx` |
| 文件类型 | TypeScript React (.tsx) |
| 行数 | 8 行 |
| 主要职责 | 提供未提交更改和每轮差异的查看界面 |

## 功能概述

该文件实现了 `/diff` 命令，用于查看 Git 未提交的更改以及每轮对话产生的差异。这是一个简单的包装组件，实际功能由 `DiffDialog` 组件实现。

## 核心内容详解

### 主要函数

**call 函数**
```typescript
export const call: LocalJSXCommandCall = async (onDone, context) => {
  const { DiffDialog } = await import('../../components/diff/DiffDialog.js');
  return <DiffDialog messages={context.messages} onDone={onDone} />;
}
```

### DiffDialog 组件

实际功能包括：
- 显示 Git 工作区的未提交更改
- 显示历史轮次的文件修改
- 支持查看特定文件的详细差异
- 提供交互式 diff 浏览

## 设计要点

1. **懒加载组件**：DiffDialog 组件动态导入
2. **消息上下文**：传递完整消息历史用于差异分析
3. **轻量级入口**：文件本身非常小

## 与其他文件的关系

- **DiffDialog.tsx**: 实际的差异查看UI组件
- **index.ts**: 命令注册
