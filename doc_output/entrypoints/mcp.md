# mcp.ts — MCP 服务器实现

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/entrypoints/mcp.ts`
- **类型**: MCP 服务器入口
- **语言**: TypeScript

## 功能概述

实现 Claude Code 的 Model Context Protocol (MCP) 服务器。允许外部客户端通过标准 MCP 协议与 Claude Code 的工具集交互。

## 核心内容详解

### 主要导出

**startMCPServer(cwd, debug, verbose)** — 启动 MCP 服务器

### MCP 服务器配置

- **服务器名称**: `claude/tengu`
- **版本**: MACRO.VERSION (构建时内联)
- **能力**: 仅工具 (tools)

### 支持的命令

```typescript
const MCP_COMMANDS: Command[] = [review]
```

### 请求处理

1. **ListTools 请求**:
   - 获取所有可用工具 (getTools)
   - 转换 Zod schema 到 JSON Schema
   - 过滤不支持的输出 schema (anyOf/oneOf 在根级别)
   - 生成工具描述

2. **CallTool 请求**:
   - 查找工具 (findToolByName)
   - 验证工具是否启用
   - 验证输入 (validateInput)
   - 调用工具 (tool.call)
   - 返回结果或错误

### 工具使用上下文 (ToolUseContext)

为非交互式会话创建简化的工具使用上下文：
- 中止控制器
- 命令列表 (仅 review)
- 主循环模型
- MCP 客户端和资源 (空)
- 应用状态存取器
- 文件状态缓存 (LRU，100 文件，25MB 限制)

### 文件状态缓存

使用有界 LRU 缓存：
- 最大文件数: 100
- 用于 readFileState 防止无界内存增长

## 设计要点

1. **工具输出 Schema 处理**:
   - MCP SDK 要求 outputSchema 在根级别有 `type: "object"`
   - 跳过在根级别有 anyOf/oneOf 的 schema
   - 来自 z.union, z.discriminatedUnion 等的 schema

2. **错误处理**:
   - 工具未找到错误
   - 工具禁用错误
   - 输入验证错误
   - 执行错误
   - 返回结构化错误响应

3. **非交互式模式**:
   - isNonInteractiveSession: true
   - 禁用思考配置
   - 空 MCP 客户端列表

4. **输入验证**:
   - 可选的 validateInput 调用
   - 支持异步验证
   - 返回验证结果和错误消息

5. **权限检查**:
   - 使用 hasPermissionsToUseTool 检查权限
   - 与交互式会话相同的权限模型

## 与其他文件的关系

- **../commands/review.js**: review 命令实现
- **../commands.js**: Command 类型定义
- **../Tool.js**: 工具查找和权限上下文
- **../tools.js**: getTools 函数
- **@modelcontextprotocol/sdk**: MCP SDK

## 注意事项

1. 输出 schema 有 MCP SDK 限制 (必须 type: object)
2. TODO: 重新暴露 MCP 工具
3. TODO: 使用 Zod 验证输入类型
4. 文件状态缓存有大小限制 (100 文件)
5. 仅支持非交互式会话模式
6. MCP 服务器使用标准输入/输出传输
