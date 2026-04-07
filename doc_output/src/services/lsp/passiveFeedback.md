# passiveFeedback.ts — LSP 被动反馈处理

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/lsp/passiveFeedback.ts`
- **所属模块**: LSP Service
- **功能类型**: 诊断通知处理

## 功能概述

该模块在所有 LSP 服务器上注册通知处理器，捕获 `textDocument/publishDiagnostics` 通知并转换为 Claude 的诊断格式。

## 核心内容详解

### 主要函数

#### `registerLSPNotificationHandlers(manager): HandlerRegistrationResult`
在所有服务器上注册诊断处理器。

**返回：**
```typescript
{
  totalServers: number        // 服务器总数
  successCount: number        // 成功注册数
  registrationErrors: Array<{ serverName, error }>  // 注册错误
  diagnosticFailures: Map<string, { count, lastError }>  // 运行时失败追踪
}
```

**流程：**
1. 遍历所有服务器（`manager.getAllServers()`）
2. 验证服务器有 `onNotification` 方法
3. 注册 `textDocument/publishDiagnostics` 处理器
4. 追踪成功/失败
5. 报告总体状态

### 诊断处理

#### 处理器逻辑
1. 验证参数（有 `uri` 和 `diagnostics`）
2. 调用 `formatDiagnosticsForAttachment` 转换格式
3. 检查是否有诊断内容
4. 注册到 `LSPDiagnosticRegistry`
5. 重置失败计数器
6. 错误时追踪失败次数（3+ 次警告）

### 格式转换

#### `formatDiagnosticsForAttachment(params): DiagnosticFile[]`
将 LSP 诊断转换为 Claude 格式。

**URI 处理：**
- `file://` URI → 文件路径（`fileURLToPath`）
- 普通路径 → 保持原样
- 失败时回退到原始 URI

**严重性映射：**
| LSP | Claude |
|-----|--------|
| 1 | Error |
| 2 | Warning |
| 3 | Info |
| 4 | Hint |
| undefined | Error |

**输出格式：**
```typescript
{
  uri: string  // 文件路径
  diagnostics: Array<{
    message: string
    severity: 'Error' | 'Warning' | 'Info' | 'Hint'
    range: { start: { line, character }, end: { line, character } }
    source?: string
    code?: string
  }>
}
```

### 失败追踪

`diagnosticFailures` Map：
- 键：服务器名称
- 值：`{ count, lastError }`

**警告阈值：**
- 3+ 连续失败时记录警告

**重置条件：**
- 成功处理诊断后清除

## 设计要点

1. **全服务器注册** — 捕获所有语言的诊断
2. **验证严格** — 无效参数被拒绝
3. **格式转换** — URI 和严重性的标准化
4. **失败追踪** — 检测持续问题
5. **错误隔离** — 单服务器失败不影响其他

## 与其他文件的关系

- **被调用**: `manager.ts` 初始化成功后
- **依赖**: `LSPDiagnosticRegistry.ts` 注册诊断
- **关联**: `diagnosticTracking.ts` 共享类型

## 注意事项

- 每个服务器独立注册，失败互不影响
- URI 转换失败时使用原始值（容错）
- 空诊断（0 项）被跳过
- 处理器内部错误被捕获，防止破坏通知循环
