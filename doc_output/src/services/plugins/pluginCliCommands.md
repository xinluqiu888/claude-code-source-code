# pluginCliCommands.ts — 插件 CLI 命令

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/services/plugins/pluginCliCommands.ts`
- **所属模块**: Plugins Service
- **功能类型**: CLI 命令包装器

## 功能概述

为插件操作提供 CLI 命令包装器，处理控制台输出、进程退出和分析事件。

## 核心内容详解

### 支持的命令

```typescript
type PluginCliCommand = 
  | 'install' 
  | 'uninstall' 
  | 'enable' 
  | 'disable' 
  | 'disable-all' 
  | 'update'
```

### 主要函数

#### `installPlugin(plugin, scope): Promise<void>`
安装插件。

**参数：**
- `plugin` — 插件标识符（名称或 plugin@marketplace）
- `scope` — 安装范围：user/project/local

**流程：**
1. 输出安装消息
2. 调用 `installPluginOp`
3. 输出成功消息
4. 记录分析事件（`_PROTO_plugin_name` 等）
5. 进程退出（0）

#### `uninstallPlugin(plugin): Promise<void>`
卸载插件。

#### `enablePlugin(plugin): Promise<void>`
启用插件。

#### `disablePlugin(plugin): Promise<void>`
禁用插件。

#### `disableAllPlugins(): Promise<void>`
禁用所有插件。

#### `updatePlugin(plugin, scope): Promise<void>`
更新插件。

### 错误处理

#### `handlePluginCommandError(error, command, plugin?): never`
统一错误处理。

**行为：**
1. 记录错误日志
2. 输出错误消息（console.error）
3. 记录分析事件 `tengu_plugin_command_failed`
4. 进程退出（1）

**遥测字段：**
- `command` — 命令类型
- `error_category` — 错误分类
- `_PROTO_plugin_name` — 插件名称（PII 标记）
- `_PROTO_marketplace_name` — 市场名称（PII 标记）

## 设计要点

1. **薄包装器** — 核心逻辑在 `pluginOperations.ts`
2. **分析集成** — 所有命令记录遥测
3. **PII 保护** — 插件名使用 `_PROTO_*` 前缀
4. **用户反馈** — 使用 `figures` 提供视觉反馈
5. **进程管理** — 显式控制退出码

## 与其他文件的关系

- **依赖**: `pluginOperations.ts` 核心操作
- **关联**: `pluginTelemetry.ts` 遥测字段构建

## 注意事项

- 成功/失败都通过 `process.exit` 退出
- 插件标识符支持 `name@marketplace` 格式
- 安装源标记为 `cli-explicit`
