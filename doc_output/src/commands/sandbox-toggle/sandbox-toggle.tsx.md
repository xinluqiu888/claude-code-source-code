# sandbox-toggle/sandbox-toggle.tsx

## 文件描述
Sandbox Toggle 组件 - 沙盒功能配置管理界面

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | React组件 |
| 导出 | call函数 |
| 功能 | 提供沙盒功能的开启/关闭界面 |
| 特性 | 支持平台检测和配置管理 |

## 函数概述

### call
主组件函数，处理沙盒功能切换：
1. 检测当前平台和环境
2. 读取当前沙盒配置
3. 显示切换界面
4. 处理用户操作

### 函数签名
```typescript
export async function call(
  onDone: LocalJSXCommandOnDone,
  context: LocalJSXCommandContext,
): Promise<React.ReactNode | null>
```

## 核心内容

### 平台检测
- 检测操作系统类型
- 检查容器环境
- 验证沙盒支持情况

### 配置管理
- 读取当前沙盒状态
- 修改配置文件
- 应用新配置

### 用户界面
- 显示当前状态
- 提供开关选项
- 显示帮助信息

## 设计点

1. **平台适配**：根据不同平台调整行为
2. **配置持久化**：修改保存到配置文件
3. **实时反馈**：操作后立即显示新状态
4. **安全验证**：确保沙盒可用时才允许开启

## 与其他文件的关系

- 使用 `isSandboxingEnabled` 读取状态
- 使用配置管理工具修改设置
- 导入 `LocalJSXCommandContext` 类型

## 注意事项

- 需要检测平台支持情况
- 某些平台可能不支持沙盒
- 配置修改需要重启生效
- 错误状态需要友好提示
