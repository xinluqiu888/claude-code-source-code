# rename/rename.ts

## 文件描述
Rename 命令实现 - 使用 AI 生成会话名称并重命名当前会话

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | Local命令 |
| 名称 | rename |
| 描述 | 重命名当前会话（AI自动生成） |
| 参数 | 可选的名称参数 |
| 模型 | claude-3-5-haiku-latest |

## 函数概述

### call
主入口函数，处理重命名逻辑：
1. 获取用户输入的名称参数
2. 如果没有提供名称，调用AI生成会话名称
3. 执行重命名操作

### 核心流程
```typescript
export const call: LocalCommandCall<RenameArgs> = async ({
  options,
  logFile,
  messages,
  abortController,
}): Promise<LocalCommandOutput>
```

## 核心内容

### 参数处理
- 获取用户提供的可选名称参数
- 支持通过命令行参数直接指定名称

### AI 命名生成
- 使用 `claude-3-5-haiku-latest` 模型
- 基于会话内容生成合适的名称
- 生成 kebab-case 格式（短横线连接）

### 重命名操作
- 更新会话文件名称
- 保持会话历史完整

## 设计点

1. **AI 驱动命名**：利用 AI 理解会话内容生成有意义名称
2. **用户可控**：允许用户直接指定名称或让AI自动生成
3. **轻量模型**：使用 Haiku 模型降低成本和提高速度
4. **格式规范**：使用 kebab-case 确保文件名兼容性

## 与其他文件的关系

- 导入 `generateSessionName` 函数生成会话名
- 使用 `LocalCommandCall` 和 `LocalCommandOutput` 类型
- 依赖 `logFile` 获取当前会话信息
- 依赖 `messages` 获取对话历史用于AI生成

## 注意事项

- 可选参数允许用户覆盖AI生成的名称
- 使用 `abortController` 支持取消操作
- 生成的名称需要确保文件系统兼容性
- 会话重命名可能影响历史记录引用
