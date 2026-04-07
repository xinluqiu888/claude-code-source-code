# controlMessageCompat.ts — 控制消息兼容性处理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/utils/controlMessageCompat.ts`
- **主要功能**: 将 camelCase 键名转换为 snake_case 以兼容旧版本 iOS
- **关键依赖**: 无

## 功能概述

该模块提供控制消息的键名规范化：
1. 将 `requestId` 转换为 `request_id`
2. 处理嵌套在 `response` 对象中的 `requestId`
3. 原地修改对象

用于修复旧版本 iOS 应用发送的控制消息（因缺少 Swift CodingKeys 映射）。

## 核心内容详解

### 问题背景

旧版本 iOS 应用发送 `requestId` 而不是 `request_id`：
- `replBridge.ts` 中的 `isSDKControlRequest` 检查 `'request_id' in value`
- `structuredIO.ts` 读取 `message.response.request_id` 为 undefined
- 导致消息被静默丢弃

### 核心函数

#### normalizeControlMessageKeys

```typescript
export function normalizeControlMessageKeys(obj: unknown): unknown
```

将 camelCase 的 `requestId` 转换为 snake_case 的 `request_id`：

1. **根级别处理**:
   - 如果对象有 `requestId` 但没有 `request_id`
   - 将 `requestId` 复制到 `request_id`
   - 删除 `requestId`

2. **嵌套 response 处理**:
   - 如果对象有 `response` 属性且是对象
   - 同样处理 `response` 对象中的 `requestId`

3. **优先级**:
   - 如果同时存在 `request_id` 和 `requestId`，snake_case 胜出

实现：
```typescript
export function normalizeControlMessageKeys(obj: unknown): unknown {
  if (obj === null || typeof obj !== 'object') return obj
  const record = obj as Record<string, unknown>
  if ('requestId' in record && !('request_id' in record)) {
    record.request_id = record.requestId
    delete record.requestId
  }
  if (
    'response' in record &&
    record.response !== null &&
    typeof record.response === 'object'
  ) {
    const response = record.response as Record<string, unknown>
    if ('requestId' in response && !('request_id' in response)) {
      response.request_id = response.requestId
      delete response.requestId
    }
  }
  return obj
}
```

## 设计要点

1. **原地修改**: 直接修改传入对象，不创建新对象
2. **安全检测**: 检查类型和属性存在性
3. **优先级处理**: snake_case 存在时保留
4. **向后兼容**: 支持新旧版本共存

## 与其他文件的关系

| 模块 | 关系 |
|------|------|
| `replBridge.ts` | 使用 `isSDKControlRequest` 检查 |
| `structuredIO.ts` | 读取 `message.response.request_id` |

## 注意事项

1. **仅处理特定键**: 只处理 `requestId`/`request_id`，不处理其他 camelCase 键
2. **iOS 特定**: 针对旧版本 iOS Swift CodingKeys 缺失问题
3. **可移除**: 当所有用户都升级后可移除此 shim
