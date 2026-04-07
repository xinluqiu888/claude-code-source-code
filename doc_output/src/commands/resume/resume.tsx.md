# resume/resume.tsx

## 文件描述
Resume 组件实现 - 提供会话恢复的用户界面

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | React组件 |
| 导出 | call函数 |
| 功能 | 显示会话选择器，恢复历史会话 |
| 组件 | LogSelector |
| 回调 | onResumeSession |

## 函数概述

### call
主组件函数，渲染会话选择器：
1. 从 context 获取 session 信息
2. 调用 onDone 回调
3. 渲染 LogSelector 组件

### 函数签名
```typescript
export async function call(
  onDone: LocalJSXCommandOnDone,
  context: LocalJSXCommandContext,
): Promise<React.ReactNode | null>
```

## 核心内容

### 组件渲染
```typescript
return (
  <LogSelector
    currentSessionId={session?.id}
    onResumeSession={onResumeSession}
    onDone={onDone}
  />
);
```

### Props 传递
| Prop | 值 | 描述 |
|------|-----|------|
| currentSessionId | session?.id | 当前会话ID |
| onResumeSession | onResumeSession | 恢复会话回调 |
| onDone | onDone | 完成回调 |

### 回调处理
- onResumeSession：处理会话恢复逻辑
- onDone：通知命令完成

## 设计点

1. **组件复用**：使用 `LogSelector` 组件显示历史会话
2. **状态传递**：传递当前会话ID避免重复选择
3. **回调分离**：区分恢复操作和完成操作
4. **异步支持**：函数设计为 async 支持异步操作

## 与其他文件的关系

- 导入 `LogSelector` 组件从 `../../components/LogSelector.js`
- 使用 `LocalJSXCommandOnDone` 和 `LocalJSXCommandContext` 类型
- 依赖 context 提供 session 和 onResumeSession

## 注意事项

- 组件立即返回，不等待用户交互
- LogSelector 负责实际的用户交互逻辑
- 当前会话ID用于高亮显示
- 需要处理无历史会话的情况
