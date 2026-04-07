# desktop.tsx — Claude Desktop 切换界面

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/desktop/desktop.tsx` |
| 文件类型 | TypeScript React (.tsx) |
| 行数 | 8 行 |
| 主要职责 | 提供切换到 Claude Desktop 应用的界面 |

## 功能概述

该文件实现了 `/desktop` 命令（别名 `/app`），允许用户将当前会话切换到 Claude Desktop 桌面应用继续。这是一个简单的包装组件，实际功能由 `DesktopHandoff` 组件实现。

## 核心内容详解

### 主要函数

**call 函数**
```typescript
export async function call(
  onDone: (result?: string, options?: { display?: CommandResultDisplay }) => void,
): Promise<React.ReactNode> {
  return <DesktopHandoff onDone={onDone} />;
}
```

### DesktopHandoff 组件

实际功能包括：
- 检测 Claude Desktop 是否已安装
- 提供打开 Desktop 应用的选项
- 传递当前会话状态到 Desktop
- 处理切换流程和错误

## 设计要点

1. **轻量级包装**：仅作为 DesktopHandoff 组件的入口
2. **别名支持**：通过 index.ts 配置 `/app` 别名
3. **平台限制**：仅在 macOS 和 Windows x64 支持

## 与其他文件的关系

- **DesktopHandoff.tsx**: 实际的切换逻辑UI
- **index.ts**: 命令注册和平台检查
