# LSPDiagnosticRegistry.ts — LSP 诊断注册表

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/lsp/LSPDiagnosticRegistry.ts`
- **所属模块**: LSP Service
- **功能类型**: 诊断数据管理和去重

## 功能概述

该模块管理从 LSP 服务器接收的诊断数据，实现异步交付模式。支持诊断注册、去重（批内和跨轮次）、容量限制和跨轮次追踪。

## 核心内容详解

### 类型定义

#### `PendingLSPDiagnostic`
```typescript
{
  serverName: string      // 发送诊断的服务器
  files: DiagnosticFile[] // 诊断文件
  timestamp: number       // 接收时间戳
  attachmentSent: boolean // 是否已发送
}
```

### 常量配置

```typescript
MAX_DIAGNOSTICS_PER_FILE = 10   // 每文件最大诊断数
MAX_TOTAL_DIAGNOSTICS = 30      // 总诊断数上限
MAX_DELIVERED_FILES = 500       // 去重追踪文件数上限
```

### 状态管理

#### `pendingDiagnostics: Map<string, PendingLSPDiagnostic>`
待处理诊断映射（UUID → 诊断）。

#### `deliveredDiagnostics: LRUCache<string, Set<string>>`
已交付诊断缓存（文件 URI → 诊断键集合）。
- LRU 策略防止无限增长
- 用于跨轮次去重

### 主要函数

#### `registerPendingLSPDiagnostic({ serverName, files }): void`
注册从服务器接收的诊断。

**流程：**
1. 生成 UUID 作为诊断 ID
2. 创建待处理诊断对象
3. 存入 `pendingDiagnostics`

#### `checkForLSPDiagnostics(): Array<{ serverName, files }>`
获取待交付的诊断（去重和限流后）。

**流程：**
1. 收集所有未发送的诊断文件
2. 执行去重（`deduplicateDiagnosticFiles`）
3. 标记已发送并从待处理中删除
4. 按严重性排序（Error < Warning < Info < Hint）
5. 应用容量限制（每文件和总数）
6. 追踪已交付诊断（用于跨轮次去重）
7. 返回聚合结果

### 去重逻辑

#### `deduplicateDiagnosticFiles(allFiles): DiagnosticFile[]`
去重诊断文件。

**去重键：**
```typescript
jsonStringify({
  message: diag.message,
  severity: diag.severity,
  range: diag.range,
  source: diag.source || null,
  code: diag.code || null,
})
```

**策略：**
- 按文件 URI 分组
- 检查是否已在当前批次中
- 检查是否已在之前轮次交付（`deliveredDiagnostics`）
- 失败时包含诊断（避免信息丢失）

### 容量限制

**每文件限制（`MAX_DIAGNOSTICS_PER_FILE = 10`）：**
- 按严重性排序（Error 优先）
- 截断超出部分

**总限制（`MAX_TOTAL_DIAGNOSTICS = 30`）：**
- 跨文件累计
- 达到上限后截断

### 状态清理

#### `clearAllLSPDiagnostics(): void`
清除所有待处理诊断。

#### `resetAllLSPDiagnosticState(): void`
重置所有状态（包括跨轮次追踪）。

#### `clearDeliveredDiagnosticsForFile(fileUri): void`
清除特定文件的已交付追踪。

**使用场景：**
- 文件编辑后，允许相同诊断重新显示

#### `getPendingLSPDiagnosticCount(): number`
获取待处理诊断数量（用于监控）。

## 设计要点

1. **异步交付模式** — 注册 → 检查 → 交付（类似 AsyncHookRegistry）
2. **双重去重** — 批内去重 + 跨轮次去重
3. LRU 缓存 — 限制内存增长
4. 错误容错 — 去重失败时保留原始诊断
5. 容量保护 — 防止大量诊断淹没系统

## 与其他文件的关系

- **被调用**: `passiveFeedback.ts` 处理 `textDocument/publishDiagnostics`
- **关联**: `diagnosticTracking.ts` 共享 `DiagnosticFile` 类型
- **使用**: 分析系统的附件交付

## 注意事项

- 诊断键使用 JSON 序列化，包含范围和内容
- 文件编辑后应调用 `clearDeliveredDiagnosticsForFile` 重置去重
- 容量限制按严重性排序后应用（Error 优先保留）
- UUID 使用 `randomUUID` 确保快速注册的唯一性
