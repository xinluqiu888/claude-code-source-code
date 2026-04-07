# builtinPlugins.ts — 内置插件注册表

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/plugins/builtinPlugins.ts`
- **类型**: TypeScript 模块
- **作用**: 管理随CLI一起分发的内置插件

## 功能概述

本模块管理内置插件（built-in plugins），这些插件随CLI一起发布，用户可以通过`/plugin`界面启用或禁用它们。内置插件与捆绑技能不同，它们可以包含多个组件（技能、钩子、MCP服务器）。

## 核心内容详解

### 内置插件标识

插件ID格式：`{name}@builtin`

与 marketplace 插件区分：`{name}@{marketplace}`

### 主要函数

#### `registerBuiltinPlugin(definition: BuiltinPluginDefinition): void`
在启动时从`initBuiltinPlugins()`调用，注册一个内置插件定义。

#### `isBuiltinPluginId(pluginId: string): boolean`
检查插件ID是否表示内置插件（以`@builtin`结尾）。

#### `getBuiltinPluginDefinition(name: string): BuiltinPluginDefinition | undefined`
获取特定内置插件的定义，供`/plugin`界面使用。

#### `getBuiltinPlugins(): { enabled: LoadedPlugin[], disabled: LoadedPlugin[] }`
获取所有内置插件，根据用户设置分为启用和禁用两组。

**启用状态优先级**：
1. 用户显式设置（`settings.enabledPlugins[pluginId]`）
2. 插件默认值（`definition.defaultEnabled`，默认为true）
3. 默认可用（true）

**可用性检查**：`isAvailable()`返回false的插件会被完全省略。

#### `getBuiltinPluginSkillCommands(): Command[]`
从启用的内置插件获取技能作为Command对象。禁用的插件技能不会被返回。

### 技能转换

```typescript
function skillDefinitionToCommand(definition: BundledSkillDefinition): Command
```

将`BundledSkillDefinition`转换为`Command`对象：
- `source`: 'bundled'（不是'builtin'，因为'builtin'在Command.source中表示硬编码的斜杠命令）
- `loadedFrom`: 'bundled'
- `isBuiltin`: 在LoadedPlugin中跟踪用户可切换特性

### 内置插件结构

```typescript
type BuiltinPluginDefinition = {
  name: string
  description: string
  version: string
  defaultEnabled?: boolean
  isAvailable?: () => boolean  // 动态可用性检查
  hooks?: HooksSettings        // 钩子配置
  mcpServers?: MCPServerConfig[] // MCP服务器配置
  skills?: BundledSkillDefinition[] // 插件提供的技能
}
```

## 设计要点

1. **与捆绑技能区分**: 内置插件出现在`/plugin` UI的"Built-in"部分，用户可开关
2. **持久化**: 启用状态保存到用户设置中
3. **多组件支持**: 一个插件可以提供技能、钩子和MCP服务器
4. **动态可用性**: 通过`isAvailable()`可以在运行时决定是否显示插件

## 与其他文件的关系

- **使用 bundledSkills.ts**: 内置插件的技能定义使用相同的`BundledSkillDefinition`类型
- **被 plugin UI 使用**: `/plugin`命令使用此模块显示和管理内置插件
- **使用 settings.ts**: 读取用户插件启用设置

## 注意事项

1. **路径哨兵**: `LoadedPlugin.path = 'builtin'`表示没有文件系统路径
2. **源字段区分**: Command.source使用'bundled'而非'builtin'，以匹配技能工具列表、分析命名和提示截断豁免的行为
3. **清理**: `clearBuiltinPlugins()`用于测试清理注册表
4. **默认启用**: 未显式设置时，插件默认为启用状态
