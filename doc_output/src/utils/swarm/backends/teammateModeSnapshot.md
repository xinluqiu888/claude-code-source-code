# teammateModeSnapshot.ts — Teammate 模式快照

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/backends/teammateModeSnapshot.ts`  
**关联模块**: 配置系统、启动流程  
**主要依赖**: `src/utils/config.js`

## 功能概述

本文件在会话启动时捕获 teammate 模式，遵循与 `hooksConfigSnapshot.ts` 相同的模式。这确保运行时配置更改不会影响当前会话的 teammate 模式。

## 核心内容详解

### 类型定义

```typescript
type TeammateMode = 'auto' | 'tmux' | 'in-process'
```

### 核心函数

#### `setCliTeammateModeOverride(mode)`
设置 CLI 覆盖的 teammate 模式：
- 必须在 `captureTeammateModeSnapshot()` 前调用
- 优先级高于配置文件

#### `getCliTeammateModeOverride()`
获取当前 CLI 覆盖（如果有），否则返回 `null`。

#### `clearCliTeammateModeOverride(newMode)`
清除 CLI 覆盖并更新快照到新模式：
- 在用户通过 UI 更改设置时调用
- 允许用户更改立即生效

#### `captureTeammateModeSnapshot()`
在会话启动时捕获 teammate 模式：
- 在 `main.tsx` 中早期调用，在 CLI 参数解析后
- CLI 覆盖优先于配置
- 从全局配置读取 `teammateMode`，默认为 `'auto'`

#### `getTeammateModeFromSnapshot()`
获取当前会话的 teammate 模式：
- 返回启动时捕获的快照
- 忽略运行时配置更改
- 如果未捕获，记录错误并回退到 `'auto'`

### 状态管理

| 状态 | 说明 |
|------|------|
| `initialTeammateMode` | 捕获的模式快照 |
| `cliTeammateModeOverride` | CLI 覆盖值 |

### 优先级

1. CLI 覆盖（`--teammate-mode`）
2. 配置文件中的 `teammateMode`
3. 默认值：`'auto'`

## 设计要点

1. **启动捕获**: 在 `main.tsx` 模块评估时捕获，早于任何生成
2. **快照模式**: 运行时配置更改不影响当前会话
3. **CLI 覆盖**: 支持 `--teammate-mode` 命令行参数
4. **用户更改支持**: `clearCliTeammateModeOverride` 允许 UI 更改生效

## 与其他文件的关系

- **config.ts**: 使用 `getGlobalConfig` 读取配置
- **main.tsx**: 调用 `captureTeammateModeSnapshot()`
- **registry.ts**: 使用 `getTeammateModeFromSnapshot()` 确定模式

## 注意事项

- 必须在 `main.tsx` 中早期调用 `captureTeammateModeSnapshot()`
- 如果调用 `getTeammateModeFromSnapshot()` 前未捕获，记录错误并自动捕获
- CLI 覆盖是一次性的，用户更改后清除
- `'auto'` 模式根据环境动态解析
