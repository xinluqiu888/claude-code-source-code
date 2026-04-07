# mailbox.tsx — 邮箱通信上下文

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/context/mailbox.tsx`
- **类型**: React Context 组件
- **语言**: TypeScript + React

## 功能概述

提供邮箱 (Mailbox) 实例的 React 上下文，用于组件间的异步消息通信。Mailbox 是一种跨组件通信机制，允许解耦的消息发送和接收。

## 核心内容详解

### 主要导出

1. **MailboxProvider** — 邮箱上下文提供者
   - 使用 useMemo 创建单例 Mailbox 实例
   - 在整个应用树中提供邮箱服务

2. **useMailbox()** — 邮箱 hook
   - 返回 Mailbox 实例
   - 必须在 MailboxProvider 内部使用，否则抛出错误

### Mailbox 类

Mailbox 类是从 `../utils/mailbox.js` 导入的，提供：
- 消息发送接口
- 消息订阅接口
- 异步消息处理

## 设计要点

1. **单例模式**:
   - 每个应用实例只有一个 Mailbox 实例
   - 使用 useMemo 确保实例在 Provider 生命周期内稳定

2. **错误边界**:
   - 如果不在 MailboxProvider 内部调用 useMailbox，抛出明确错误
   - 错误消息: "useMailbox must be used within a MailboxProvider"

3. **React Compiler 优化**:
   - 使用 `_c` 函数进行编译器优化
   - 使用 Symbol.for("react.memo_cache_sentinel") 作为缓存标记

## 与其他文件的关系

- **../utils/mailbox.js**: Mailbox 类的实现
- **应用根组件**: 在顶层渲染 MailboxProvider 包裹应用

## 注意事项

1. Mailbox 实例在 Provider 创建时初始化
2. 适用于跨组件的异步消息通信
3. 与 Redux/Zustand 等状态管理不同，Mailbox 专注于消息传递
4. 必须在 Provider 内部使用 useMailbox hook
