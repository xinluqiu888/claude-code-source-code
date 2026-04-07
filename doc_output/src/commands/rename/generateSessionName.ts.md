# rename/generateSessionName.ts

## 文件描述
会话名称生成器 - 使用 AI 生成基于对话内容的会话名称

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | 工具函数 |
| 导出 | generateSessionName |
| 功能 | 基于消息历史生成会话名称 |
| 模型 | claude-3-5-haiku-latest |
| 输出 | kebab-case 格式字符串 |

## 函数概述

### generateSessionName
异步函数，基于对话内容生成合适的会话名称：
1. 分析消息历史
2. 提取关键主题
3. 生成简洁的描述性名称
4. 格式化为 kebab-case

### 函数签名
```typescript
export async function generateSessionName(
  messages: Message[],
  abortController: AbortController,
): Promise<string>
```

## 核心内容

### 输入参数
| 参数 | 类型 | 描述 |
|------|------|------|
| messages | Message[] | 会话消息历史 |
| abortController | AbortController | 用于取消操作 |

### 输出格式
- 使用 kebab-case（短横线连接的小写字母）
- 例如：`fix-login-bug`, `add-user-auth`, `refactor-database`

### AI 提示策略
- 分析最后几条消息理解当前主题
- 提取关键动作和对象
- 生成简洁、有意义的名称
- 限制名称长度避免过长

## 设计点

1. **内容感知**：基于实际对话内容生成相关名称
2. **格式标准化**：使用 kebab-case 确保跨平台兼容
3. **轻量模型**：Haiku 模型提供快速、经济的推理
4. **可中断**：支持通过 AbortController 取消生成

## 与其他文件的关系

- 被 `rename.ts` 调用用于生成默认会话名
- 使用 `Message` 类型定义消息结构
- 集成 AI 模型 API 进行名称生成

## 注意事项

- 名称生成是异步操作，可能需要等待
- 结果需要经过清理确保文件系统安全
- 应考虑名称唯一性避免冲突
- 生成失败时应有合理的回退机制
