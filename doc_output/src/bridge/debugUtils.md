# debugUtils.ts — 桥接调试工具函数

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/bridge/debugUtils.ts`
- **文件类型**: TypeScript 模块
- **行数**: 约 142 行
- **主要职责**: 提供桥接层的调试工具函数，包括敏感信息脱敏、错误描述提取和分析事件记录

## 功能概述

`debugUtils.ts` 是桥接层的调试工具模块，提供安全日志记录、错误处理和分析功能。模块的核心关注点是安全性和可调试性——在记录详细信息的同时避免泄露敏感凭证。

主要功能包括：字符串截断和脱敏处理，防止日志中泄露访问令牌、会话密钥等敏感信息；axios 错误描述提取，包括 HTTP 状态码和服务器错误详情；以及桥接初始化跳过的统一日志记录。

模块被多个桥接相关文件使用，提供一致的调试输出格式和安全处理策略。

## 核心内容详解

### 常量定义

| 常量 | 值 | 说明 |
|------|-----|------|
| `DEBUG_MSG_LIMIT` | 2000 | 调试消息截断长度限制 |
| `SECRET_FIELD_NAMES` | 数组 | 需要脱敏的字段名列表 |
| `REDACT_MIN_LENGTH` | 16 | 最小脱敏长度（短值完全脱敏） |

**敏感字段列表**:
- `session_ingress_token`
- `environment_secret`
- `access_token`
- `secret`
- `token`

### redactSecrets 函数

**签名**: `redactSecrets(s: string): string`

**功能**: 脱敏字符串中的敏感字段值。

**算法**:
1. 使用正则表达式匹配 `"{field}"\s*:\s*"{value}"` 模式
2. 字段名来自 `SECRET_FIELD_NAMES`
3. 如果值长度 < 16，完全脱敏为 `[REDACTED]`
4. 如果值长度 >= 16，显示前 8 字符和最后 4 字符，中间用 `...` 替代

**示例**:
```typescript
// 输入
'{"access_token":"abcdef1234567890xyz","secret":"short"}'

// 输出
'{"access_token":"abcdef12...xyz","secret":"[REDACTED]"}'
```

### debugTruncate 函数

**签名**: `debugTruncate(s: string): string`

**功能**: 截断字符串用于调试日志，将换行符折叠为 `\n`。

**行为**:
- 将 `\n` 替换为 `\n`（字面量）
- 如果长度 <= 2000，返回原字符串
- 否则截断到 2000 字符并追加 `... ({total} chars)`

### debugBody 函数

**签名**: `debugBody(data: unknown): string`

**功能**: 截断可 JSON 序列化的值用于调试。

**流程**:
1. 如果输入是字符串，直接使用
2. 否则序列化为 JSON
3. 应用 `redactSecrets`
4. 如果长度 <= 2000，返回结果
5. 否则截断并追加字符计数

### describeAxiosError 函数

**签名**: `describeAxiosError(err: unknown): string`

**功能**: 从 axios 错误中提取描述性错误消息。

**增强逻辑**:
1. 获取基础错误消息
2. 如果错误有 `response.data`：
   - 优先检查 `data.message`
   - 其次检查 `data.error.message`
   - 如果找到，追加到基础消息

**用途**: 默认 axios 消息只包含状态码，此函数添加服务器返回的详细错误信息。

### extractHttpStatus 函数

**签名**: `extractHttpStatus(err: unknown): number | undefined`

**功能**: 从 axios 错误中提取 HTTP 状态码。

**行为**: 检查 `err.response.status` 是否为数字，是则返回，否则返回 `undefined`。用于区分 HTTP 错误（有状态码）和网络错误（无状态码）。

### extractErrorDetail 函数

**签名**: `extractErrorDetail(data: unknown): string | undefined`

**功能**: 从 API 错误响应体中提取人类可读的消息。

**检查顺序**:
1. `data.message`（字符串）
2. `data.error.message`（字符串）

这是 `describeAxiosError` 使用的底层函数，也可直接用于响应体数据。

### logBridgeSkip 函数

**签名**: `logBridgeSkip(reason: string, debugMsg?: string, v2?: boolean): void`

**功能**: 记录桥接初始化跳过事件。

**行为**:
1. 如果提供 `debugMsg`，记录调试日志
2. 记录 `tengu_bridge_repl_skipped` 分析事件
3. 包含原因和可选的 v2 标志

**用途**: 统一桥接跳过事件的日志和分析记录，避免调用点重复样板代码。

## 设计要点

1. **深度防御**: `redactSecrets` 使用正则表达式而非 JSON 解析，可以处理部分 JSON 或格式化的输出。

2. **分级脱敏**: 短值（<16）完全脱敏，长值保留前缀和后缀以便调试识别。

3. **换行折叠**: `debugTruncate` 将换行转为字面量 `\n`，使日志保持单行，便于 grep 和过滤。

4. **错误信息增强**: `describeAxiosError` 提取服务器错误详情，比默认的 "Request failed with status code 400" 更有用。

5. **类型安全**: 所有提取函数使用 `unknown` 输入，内部进行类型守卫检查，避免运行时崩溃。

6. **统一事件**: `logBridgeSkip` 集中处理跳过事件，确保分析数据一致性。

7. **长度限制**: 2000 字符限制防止超大响应（如二进制数据意外序列化）淹没日志。

## 与其他文件的关系

| 文件 | 关系描述 |
|------|----------|
| `../services/analytics/index.js` | 导入 `logEvent` 和 `AnalyticsMetadata_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS` |
| `../utils/debug.js` | 导入 `logForDebugging` |
| `../utils/errors.js` | 导入 `errorMessage` |
| `../utils/slowOperations.js` | 导入 `jsonStringify` |
| `codeSessionApi.ts` | 使用 `extractErrorDetail` 提取 API 错误 |
| `createSession.ts` | 使用 `extractErrorDetail` 提取 Sessions API 错误 |
| `replBridge.ts` / `bridgeMain.ts` | 使用 `logBridgeSkip` 记录跳过事件 |

## 注意事项

1. **正则局限性**: `redactSecrets` 的正则 `"field"\s*:\s*"value"` 假设 JSON 格式，可能遗漏非标准格式（如单引号、无引号键）。

2. **Unicode 长度**: `debugTruncate` 使用 JavaScript 字符串长度（码元计数），而非视觉宽度或 grapheme 计数。emoji 等可能计算不准确。

3. **递归数据**: `jsonStringify` 可能因循环引用抛出，但模块未显式处理此情况。

4. **状态码范围**: `extractHttpStatus` 只检查 `typeof === 'number'`，不验证范围（如 100-599）。

5. **错误详情嵌套**: `extractErrorDetail` 只检查一级 `message` 和二级 `error.message`，更深嵌套被忽略。

6. **敏感字段扩展**: 添加新敏感字段需要修改 `SECRET_FIELD_NAMES` 数组并重新部署。

7. **调试级别**: `logForDebugging` 的日志级别由全局调试配置控制，模块不控制输出级别。

8. **AnalyticsMetadata 类型**: 使用特殊命名的类型 `AnalyticsMetadata_I_VERIFIED_THIS_IS_NOT_CODE_OR_FILEPATHS` 强制代码审查者确认元数据安全。
