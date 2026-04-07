# mcpSkillBuilders.ts — MCP技能构建器注册模块

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/skills/mcpSkillBuilders.ts`
- **类型**: TypeScript 模块
- **作用**: 提供MCP技能发现所需的`loadSkillsDir`函数的写一次注册表

## 功能概述

本模块是一个依赖图叶子节点，专门用于解决循环依赖问题。它通过提供一个注册表模式，让`mcpSkills.ts`和`loadSkillsDir.ts`可以相互依赖而不会形成循环依赖（client.ts → mcpSkills.ts → loadSkillsDir.ts → ... → client.ts）。

## 核心内容详解

### 类型定义

```typescript
export type MCPSkillBuilders = {
  createSkillCommand: typeof createSkillCommand
  parseSkillFrontmatterFields: typeof parseSkillFrontmatterFields
}
```

### 导出函数

1. **`registerMCPSkillBuilders(b: MCPSkillBuilders): void`**
   - 注册MCP技能构建器函数
   - 在`loadSkillsDir.ts`模块初始化时调用

2. **`getMCPSkillBuilders(): MCPSkillBuilders`**
   - 获取已注册的技能构建器
   - 如果尚未注册会抛出错误

### 设计决策

- **非字面量动态导入问题**: `"await import(variable)"`在Bun打包的二进制文件中会失败，因为路径解析基于chunk的`/$bunfs/root/...`路径而非原始源树
- **字面量动态导入问题**: 虽然能工作，但dependency-cruiser会跟踪它，导致循环依赖检查失败
- **解决方案**: 使用此注册表模块作为中介，避免直接的模块间依赖

## 设计要点

1. **依赖图叶子节点**: 本模块只导入类型，不导入实际模块，因此不会形成循环
2. **写一次注册**: 在`loadSkillsDir.ts`模块初始化时完成注册，早于任何MCP服务器连接
3. **运行时安全**: 使用Bun兼容的模块解析方式

## 与其他文件的关系

- **被 loadSkillsDir.ts 导入**: 在模块初始化时调用`registerMCPSkillBuilders`
- **被 mcpSkills.ts 使用**: 通过`getMCPSkillBuilders`获取构建器函数
- **依赖类型**: 仅依赖`./loadSkillsDir.js`的类型定义

## 注意事项

- 注册必须在任何MCP服务器连接之前完成
- 该模块是启动流程的关键部分，通过`commands.ts`的静态导入在启动时立即求值
- 错误提示：如果调用`getMCPSkillBuilders`时还未注册，会抛出"loadSkillsDir.ts has not been evaluated yet"错误
