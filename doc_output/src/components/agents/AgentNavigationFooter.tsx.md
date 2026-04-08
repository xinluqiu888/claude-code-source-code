# AgentNavigationFooter.tsx — Agent导航底部栏

> **一句话总结**：显示Agent界面的操作提示和退出状态信息。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/components/agents/AgentNavigationFooter.tsx` |
| 文件类型 | React组件 (TSX) |
| 主要职责 | 显示导航提示和操作说明 |

---

## 功能概述

`AgentNavigationFooter`是一个简单的展示组件，用于在Agent相关界面底部显示操作提示。它显示：
- 导航操作说明（上下箭头导航、回车选择、Esc返回）
- 退出确认状态（当用户按下Ctrl+C/D时）

---

## 核心内容详解

### 主要类型

- **Props**:
  - `instructions?`: 自定义操作说明文本（可选）

### 主要函数

- **AgentNavigationFooter**: 主组件函数
  - **默认提示**: "Press ↑↓ to navigate · Enter to select · Esc to go back"
  - **退出状态**: 通过`useExitOnCtrlCDWithKeybindings`获取
  - **条件显示**: 当exitState.pending为true时显示退出确认提示

### 渲染逻辑

1. 使用`useExitOnCtrlCDWithKeybindings`监听退出快捷键
2. 如果有退出请求正在等待确认，显示"Press [key] again to exit"
3. 否则显示默认或自定义的操作说明
4. 使用dimColor样式使提示文字呈淡化效果

---

## 设计要点

- 简洁单一职责，只负责显示提示信息
- 支持自定义提示内容
- 与退出快捷键系统无缝集成
- 使用React Compiler优化

---

## 与其他文件的关系

- **依赖**:
  - `useExitOnCtrlCDWithKeybindings` - 退出快捷键状态
  - `Box`, `Text` - Ink UI组件
- **被依赖**:
  - `AgentsMenu` - 作为底部导航栏使用

---

## 注意事项

- 提示文字使用淡化颜色，避免分散用户注意力
- 退出确认状态优先级高于普通提示
