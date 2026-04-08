# it2Setup.ts — iTerm2 设置模块

## 基本信息

**文件路径**: `/root/projects/claude-code-source-code/src/utils/swarm/backends/it2Setup.ts`  
**关联模块**: iTerm2 集成、CLI 工具安装  
**主要依赖**: `src/utils/execFileNoThrow.js`, `src/utils/config.js`

## 功能概述

本文件处理 it2 CLI 工具的安装和设置：
- 检测 Python 包管理器（uv、pipx、pip）
- 安装 it2 CLI 工具
- 验证 it2 设置
- 管理用户首选项

## 核心内容详解

### 类型定义

```typescript
type PythonPackageManager = 'uvx' | 'pipx' | 'pip'

type It2InstallResult = {
  success: boolean
  error?: string
  packageManager?: PythonPackageManager
}

type It2VerifyResult = {
  success: boolean
  error?: string
  needsPythonApiEnabled?: boolean
}
```

### 核心函数

#### `detectPythonPackageManager(): Promise<PythonPackageManager | null>`
检测系统可用的 Python 包管理器：
- 优先顺序：uv → pipx → pip/pip3
- 使用 `which` 命令检查
- 返回检测到的管理器或 `null`

#### `isIt2CliAvailable(): Promise<boolean>`
检查 it2 CLI 是否已安装：
- 使用 `which it2` 检查

#### `installIt2(packageManager): Promise<It2InstallResult>`
使用检测到的包管理器安装 it2：
- 从家目录运行以避免读取项目级配置（pip.conf/uv.toml）
- 支持 uv：`uv tool install it2`
- 支持 pipx：`pipx install it2`
- 支持 pip：`pip install --user it2`（失败时尝试 pip3）

#### `verifyIt2Setup(): Promise<It2VerifyResult>`
验证 it2 是否正确配置：
- 首先检查 it2 是否安装
- 运行 `it2 session list` 测试 Python API 连接
- 检测常见错误（API 未启用、连接被拒绝）

#### `getPythonApiInstructions(): string[]`
返回启用 iTerm2 Python API 的说明：
```
Almost done! Enable the Python API in iTerm2:

  iTerm2 → Settings → General → Magic → Enable Python API

After enabling, you may need to restart iTerm2.
```

#### `markIt2SetupComplete()`
标记 it2 设置已完成：
- 设置 `iterm2It2SetupComplete: true` 到全局配置
- 防止再次显示设置提示

#### `setPreferTmuxOverIterm2(prefer)`
设置用户是否偏好 tmux 而非 iTerm2：
- 设置 `preferTmuxOverIterm2` 到全局配置
- 防止在 iTerm2 中显示设置提示

#### `getPreferTmuxOverIterm2(): boolean`
检查用户是否偏好 tmux 而非 iTerm2。

## 设计要点

1. **家目录安装**: 避免项目级 pip.conf/uv.toml 可能的安全风险
2. **多管理器支持**: 优先 uv，回退到 pipx/pip
3. **功能测试**: 验证使用 `session list` 而非 `version`（后者即使 API 禁用时也成功）
4. **用户首选项**: 允许用户选择偏好 tmux 并持久化

## 与其他文件的关系

- **registry.ts**: 使用 `getPreferTmuxOverIterm2` 检测
- **config.ts**: 使用 `saveGlobalConfig` 持久化首选项
- **detection.ts**: `isIt2CliAvailable` 被用于检测

## 注意事项

- 必须从家目录运行安装以避免项目级配置
- Python API 必须在 iTerm2 设置中手动启用
- `verifyIt2Setup` 检测 API 未启用错误并提供说明
- 设置完成后标记，避免重复提示
