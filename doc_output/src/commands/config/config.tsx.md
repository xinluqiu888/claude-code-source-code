# config.tsx — 配置面板界面

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/config/config.tsx` |
| 文件类型 | TypeScript React (.tsx) |
| 行数 | 6 行 |
| 主要职责 | 提供配置面板设置界面入口 |

## 功能概述

该文件实现了 `/config` 命令（别名 `/settings`），用于打开 Claude Code 的配置面板。这是一个简单的包装组件，实际功能由 `Settings` 组件实现。

## 核心内容详解

### 主要函数

**call 函数**
```typescript
export const call: LocalJSXCommandCall = async (onDone, context) => {
  return <Settings onClose={onDone} context={context} defaultTab="Config" />;
}
```

### Settings 组件

实际功能包括：
- 显示各种配置选项
- 支持多个标签页（Config、其他设置）
- 保存用户偏好设置
- 管理本地和全局配置

## 设计要点

1. **默认标签页**：打开时默认显示 "Config" 标签
2. **上下文传递**：传递完整的命令上下文给 Settings
3. **回调处理**：使用 `onClose` 处理关闭事件

## 与其他文件的关系

- **Settings/Settings.tsx**: 实际的配置面板实现
- **index.ts**: 命令注册
