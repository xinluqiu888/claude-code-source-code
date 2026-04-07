# LSPTool.ts — LSP协议代码智能工具

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/LSPTool/LSPTool.ts`
- **工具名称**: `LSP`
- **类型**: 代码智能分析工具

## 功能概述

LSPTool是与Language Server Protocol (LSP)服务器交互的工具，提供代码智能功能，如跳转到定义、查找引用、悬停提示、符号搜索等。支持9种LSP操作，帮助开发者理解代码结构和关系。

## 核心内容详解

### 支持的LSP操作

1. **goToDefinition**: 查找符号定义位置
2. **findReferences**: 查找所有引用
3. **hover**: 获取悬停信息（文档、类型）
4. **documentSymbol**: 获取文档内所有符号
5. **workspaceSymbol**: 跨工作区搜索符号
6. **goToImplementation**: 查找接口实现
7. **prepareCallHierarchy**: 准备调用层次结构
8. **incomingCalls**: 查找调用者
9. **outgoingCalls**: 查找被调用者

### 主要功能

```typescript
// 等待LSP初始化完成
const status = getInitializationStatus()
if (status.status === 'pending') {
  await waitForInitialization()
}

// 确保文件在LSP服务器中打开
if (!manager.isFileOpen(absolutePath)) {
  const fileContent = await handle.readFile({ encoding: 'utf-8' })
  await manager.openFile(absolutePath, fileContent)
}

// 发送LSP请求
let result = await manager.sendRequest(absolutePath, method, params)
```

### 安全特性

- **UNC路径防护**: 跳过UNC路径的文件系统操作，防止NTLM凭据泄漏
- **文件大小限制**: 最大10MB文件限制
- **Git忽略过滤**: 自动过滤gitignore中的文件

### 调用层次处理

对于`incomingCalls`和`outgoingCalls`，采用两步流程：
1. 首先通过`prepareCallHierarchy`获取CallHierarchyItem
2. 然后使用该item请求实际的调用信息

## 设计要点

1. **延迟执行**: `shouldDefer: true`，确保LSP服务器准备就绪
2. **并发安全**: `isConcurrencySafe: true`
3. **只读操作**: `isReadOnly: true`
4. **结果格式化**: 每种操作有专门的格式化器，位于`formatters.ts`

## 与其他文件的关系

- **prompt.ts**: 工具描述和提示文本
- **schemas.ts**: Zod输入验证模式
- **formatters.ts**: 结果格式化函数
- **UI.tsx**: 渲染函数
- **symbolContext.ts**: 符号提取辅助函数
- **services/lsp/manager.ts**: LSP服务器管理

## 注意事项

- LSP服务器必须针对文件类型配置
- 使用1-based行列号（与编辑器一致）
- 内部转换为0-based（LSP协议要求）
- 结果自动过滤gitignore文件
