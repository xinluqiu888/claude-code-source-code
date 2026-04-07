# passes/passes.tsx

## 文件描述
Guest Passes 组件 - 处理访客通行证的UI展示和交互

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | React组件 |
| 导出 | call函数 |
| 功能 | 打开浏览器到访客通行证页面 |
| 交互 | 展示登录界面用于重新登录 |

## 函数概述

### call
异步函数，处理访客通行证逻辑：
1. 打开浏览器访问 `https://claude.ai/passes`
2. 显示登录界面
3. 登录成功后触发API密钥变更回调

## 核心内容

### 组件实现
```typescript
export async function call(
  onDone: LocalJSXCommandOnDone,
  context: LocalJSXCommandContext,
): Promise<React.ReactNode | null>
```

### 关键行为
- 打开浏览器到 `https://claude.ai/passes`
- 返回 Login 组件用于重新登录
- 登录成功后调用 `context.onChangeAPIKey()`

## 设计点

1. **浏览器集成**：使用 `openBrowser` 工具打开外部链接
2. **登录流程**：集成登录组件处理身份验证
3. **状态同步**：登录成功后通知系统API密钥已变更

## 与其他文件的关系

- 导入 `Login` 组件从 `../login/login.js`
- 使用 `openBrowser` 从 `../../utils/browser.js`
- 使用 `LocalJSXCommandContext` 和 `LocalJSXCommandOnDone` 类型

## 注意事项

- 使用 `React.Fragment` 作为容器
- 登录消息提示用户可以使用 Ctrl-C 退出
- 需要处理登录成功和失败的不同状态
