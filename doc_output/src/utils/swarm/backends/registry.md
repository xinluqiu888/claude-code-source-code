# registry.ts — 后端注册表

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/backends/registry.ts`  
**关联模块**: 后端检测、队友执行  
**主要依赖**: `src/bootstrap/state.js`, `src/utils/platform.js`

## 功能概述

本文件提供后端注册、检测和管理的中央注册表：
- 后端类的注册和实例化
- 自动后端检测（优先级：tmux > iTerm2 > tmux 外部）
- 获取 `TeammateExecutor` 的统一接口
- in-process 模式启用检查

## 核心内容详解

### 后端检测优先级

1. **在 tmux 中**: 始终使用 tmux
2. **在 iTerm2 中，it2 可用**: 使用 iTerm2 原生窗格
3. **在 iTerm2 中，it2 不可用，tmux 可用**: 使用 tmux 作为回退
4. **不在 tmux/iTerm2 中，tmux 可用**: 使用 tmux 外部会话
5. **无后端可用**: 抛出错误

### 核心函数

#### `ensureBackendsRegistered()`
确保后端类已动态导入，使 `getBackendByType()` 可以构造它们。
- 轻量级选项，仅用于类注册（如按 backendType 杀死窗格）
- 与 `detectAndGetBackend()` 不同，从不生成子进程

#### `detectAndGetBackend(): Promise<BackendDetectionResult>`
检测并返回适当的窗格后端：
- 使用缓存结果（一旦检测，固定不变）
- 返回后端实例、是否原生、是否需要 it2 设置

#### `isInProcessEnabled(): boolean`
检查是否启用了 in-process teammate 执行：
- 非交互式会话强制启用
- `teammateMode: 'in-process'` 始终启用
- `teammateMode: 'tmux'` 始终禁用
- `teammateMode: 'auto'` 检查环境：在 tmux/iTerm2 中使用 pane，否则 in-process

#### `getTeammateExecutor(preferInProcess?): Promise<TeammateExecutor>`
获取用于生成 teammates 的 `TeammateExecutor`：
- 如果 `preferInProcess` 为 true 且 in-process 启用，返回 `InProcessBackend`
- 否则返回 `PaneBackendExecutor` 包装检测到的窗格后端

#### `getInProcessBackend(): TeammateExecutor`
获取缓存的 `InProcessBackend` 实例，首次调用时创建。

#### `getResolvedTeammateMode(): 'in-process' | 'tmux'`
返回当前会话解析后的 teammate 模式（'auto' 解析为实际值）。

### 平台特定的 tmux 安装说明

```typescript
function getTmuxInstallInstructions(): string
```

- **macOS**: `brew install tmux`
- **Linux/WSL**: `sudo apt install tmux` 或 `sudo dnf install tmux`
- **Windows**: 需要 WSL，然后在 WSL 中安装 tmux

### 状态管理

| 状态 | 说明 |
|------|------|
| `cachedBackend` | 缓存的后端实例 |
| `cachedDetectionResult` | 缓存的检测结果 |
| `backendsRegistered` | 后端类是否已注册 |
| `inProcessFallbackActive` | 是否已回退到 in-process 模式 |

## 设计要点

1. **延迟导入**: 后端类通过 `import()` 动态加载，避免循环依赖
2. **自注册**: 后端模块导入时调用 `registerTmuxBackend`/`registerITermBackend`
3. **单例模式**: 缓存后端实例避免重复创建
4. **回退机制**: 记录 pane 后端不可用时回退到 in-process

## 与其他文件的关系

- **TmuxBackend.ts**: 注册 tmux 后端类
- **ITermBackend.ts**: 注册 iTerm2 后端类
- **InProcessBackend.ts**: 创建 in-process 后端实例
- **teammateModeSnapshot.ts**: 获取 teammate 模式

## 注意事项

- `detectAndGetBackend()` 可能抛出错误（tmux 未安装）
- `isInsideTmux()` 和 `isInITerm2()` 用于环境检测
- 一旦 `inProcessFallbackActive` 设置，后续生成将短路到 in-process
- 测试使用 `resetBackendDetection()` 重置状态
