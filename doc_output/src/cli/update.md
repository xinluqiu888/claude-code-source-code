# update.ts — CLI 自动更新功能

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/update.ts` |
| **文件类型** | TypeScript 功能模块 |
| **行数** | 422 行 |
| **职责** | 实现 `claude update` 命令，支持多种安装方式的自动更新检测和安装 |

## 功能概述

本模块实现 Claude Code CLI 的自动更新功能，支持检测新版本并执行更新。它处理多种安装场景：
- **原生安装（native）**：使用原生安装器进行更新
- **npm 本地安装（npm-local）**：更新本地 node_modules
- **npm 全局安装（npm-global）**：更新全局 npm 包
- **包管理器安装**：Homebrew、winget、apk 等（仅提示用户手动更新）
- **开发构建（development）**：拒绝更新并提示警告

模块首先运行诊断检测当前安装类型，然后根据安装类型选择合适的更新策略，处理版本比较、权限检查、配置同步等。

## 核心内容详解

### 导入

| 导入项 | 来源 | 用途 |
|--------|------|------|
| `chalk` | `chalk` | 终端彩色输出 |
| `logEvent` | `src/services/analytics` | 分析事件记录 |
| `getLatestVersion`, `installGlobalPackage` | `src/utils/autoUpdater.js` | 自动更新工具 |
| `getGlobalConfig`, `saveGlobalConfig` | `src/utils/config.js` | 全局配置管理 |
| `logForDebugging` | `src/utils/debug.js` | 调试日志 |
| `getDoctorDiagnostic` | `src/utils/doctorDiagnostic.js` | 运行安装诊断 |
| `gracefulShutdown` | `src/utils/gracefulShutdown.js` | 优雅关闭 |
| `installOrUpdateClaudePackage` | `src/utils/localInstaller.js` | 本地安装更新 |
| `installLatest as installLatestNative` | `src/utils/nativeInstaller` | 原生安装更新 |
| `writeToStdout` | `src/utils/process.js` | 标准输出写入 |
| `gte` | `src/utils/semver.js` | 语义版本比较 |

### 主函数

#### `export async function update()`

更新功能的主入口函数。

**执行流程**：

1. **日志和初始化**
   - 记录 `tengu_update_check` 分析事件
   - 显示当前版本
   - 获取更新通道（latest/stable）

2. **运行诊断**
   - 调用 `getDoctorDiagnostic()` 检测安装类型
   - 检查多个安装
   - 显示警告（如 PATH 问题）

3. **配置更新**
   - 如果未设置 installMethod，根据检测到的安装类型自动设置
   - 映射诊断安装类型到配置安装方法：
     - `npm-local` -> `local`
     - `native` -> `native`
     - `npm-global` -> `global`

4. **特殊情况处理**
   - **开发构建**：拒绝更新并退出（错误码 1）
   - **包管理器安装**：提示用户使用对应包管理器命令更新

5. **配置/实际不匹配检测**
   - 检测配置文件期望的安装方式与实际运行的安装方式是否一致
   - 如不一致，更新配置并提示用户

6. **原生安装更新**
   - 调用 `installLatestNative()`
   - 处理锁竞争（lock contention）
   - 显示更新结果

7. **npm 安装更新**
   - 如果不是原生安装，移除原生安装符号链接
   - 从 npm registry 获取最新版本
   - 根据安装类型（本地/全局）选择更新方法
   - 处理更新结果状态

**错误处理**：
- 网络问题：提供详细的故障排除建议
- 权限问题：提示使用 sudo 或修复 npm 权限
- 安装失败：提示手动更新命令

### 状态处理

#### `InstallStatus` 类型

| 状态值 | 含义 |
|--------|------|
| `success` | 更新成功 |
| `no_permissions` | 权限不足 |
| `install_failed` | 安装失败 |
| `in_progress` | 另一个实例正在执行更新 |

### 关键设计决策

1. **诊断优先**：总是先运行诊断来确定安装类型，而不是信任配置文件
2. **配置同步**：自动检测并修复配置与实际安装类型不匹配的情况
3. **包管理器检测**：识别 Homebrew、winget、apk 等包管理器，并引导用户使用正确的命令
4. **锁处理**：原生安装更新时优雅处理锁竞争，显示持有锁的进程信息
5. **版本比较**：使用语义版本比较（`gte`），支持预发布版本和构建元数据

## 设计要点

1. **多安装类型支持**：全面支持原生、npm 本地、npm 全局、包管理器、开发构建等多种安装方式
2. **用户体验**：提供清晰的警告和错误信息，包含修复建议
3. **权限处理**：区分权限不足和安装失败，提供针对性的解决建议
4. **配置管理**：自动维护 installMethod 配置，确保与实际安装类型一致
5. **分析集成**：记录关键事件（开始检查、成功、失败）用于产品分析
6. **诊断复用**：复用 `doctor` 命令的诊断逻辑，确保一致性

## 与其他文件的关系

| 关系类型 | 文件 | 描述 |
|---------|------|------|
| **导入** | `src/utils/autoUpdater.js` | 获取最新版本、全局安装 |
| **导入** | `src/utils/doctorDiagnostic.js` | 运行安装诊断 |
| **导入** | `src/utils/localInstaller.js` | 本地 npm 安装更新 |
| **导入** | `src/utils/nativeInstaller/index.js` | 原生安装更新 |
| **导入** | `src/utils/config.js` | 全局配置读写 |
| **导入** | `src/services/analytics` | 分析事件记录 |
| **导入** | `src/utils/gracefulShutdown.js` | 优雅关闭 |
| **被导入** | `src/main.tsx` | `claude update` 命令处理 |

## 注意事项

1. **开发构建检测**：明确拒绝更新开发构建，防止意外行为
2. **包管理器特殊处理**：对于 pacman、deb、rpm 等没有单一前端命令的包管理器，只提供通用提示
3. **环境变量检查**：`ANTHROPIC_API_KEY` 仅在非 homespace 环境被视为有效 API key 源
4. **版本字符串**：使用 `MACRO.VERSION` 和 `MACRO.PACKAGE_URL` 编译时常量
5. **网络故障排除**：npm registry 失败时提供详细的网络故障排除建议
6. **完成缓存刷新**：成功更新后调用 `regenerateCompletionCache()` 刷新补全缓存
