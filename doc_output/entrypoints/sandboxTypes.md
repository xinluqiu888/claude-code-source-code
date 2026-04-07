# sandboxTypes.ts — 沙箱配置类型

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/entrypoints/sandboxTypes.ts`
- **类型**: TypeScript 类型定义
- **语言**: TypeScript

## 功能概述

Claude Code Agent SDK 沙箱配置的单一真相源。包含网络配置、文件系统配置和沙箱设置的 Zod schema 定义，供 SDK 和设置验证共同使用。

## 核心内容详解

### Schema 导出

1. **SandboxNetworkConfigSchema** — 网络配置
   - `allowedDomains`: 允许的域名数组
   - `allowManagedDomainsOnly`: 仅允许托管域名
   - `allowUnixSockets`: Unix socket 路径 (macOS 专用)
   - `allowAllUnixSockets`: 允许所有 Unix sockets
   - `allowLocalBinding`: 允许本地绑定
   - `httpProxyPort`: HTTP 代理端口
   - `socksProxyPort`: SOCKS 代理端口

2. **SandboxFilesystemConfigSchema** — 文件系统配置
   - `allowWrite`: 额外允许写入的路径
   - `denyWrite`: 额外拒绝写入的路径
   - `denyRead`: 额外拒绝读取的路径
   - `allowRead`: 重新允许读取的路径
   - `allowManagedReadPathsOnly`: 仅允许托管读取路径

3. **SandboxSettingsSchema** — 沙箱设置
   - `enabled`: 是否启用
   - `failIfUnavailable`: 不可用时退出
   - `autoAllowBashIfSandboxed`: 沙箱化时自动允许 Bash
   - `allowUnsandboxedCommands`: 允许非沙箱命令
   - `network`: 网络配置
   - `filesystem`: 文件系统配置
   - `ignoreViolations`: 忽略违规记录
   - `enableWeakerNestedSandbox`: 启用弱嵌套沙箱
   - `enableWeakerNetworkIsolation`: 启用弱网络隔离 (macOS)
   - `excludedCommands`: 排除的命令
   - `ripgrep`: ripgrep 自定义配置

### 类型导出

从 schema 推断的类型：
- `SandboxSettings`
- `SandboxNetworkConfig`
- `SandboxFilesystemConfig`
- `SandboxIgnoreViolations`

### 关键特性

1. **延迟 Schema**:
   - 使用 `lazySchema` 包装
   - 避免循环依赖
   - 支持运行时 schema 创建

2. **安全注释**:
   - 详细描述每个选项的安全影响
   - `enableWeakerNetworkIsolation` 警告数据泄露风险
   - `allowUnsandboxedCommands` 说明

3. **平台特定选项**:
   - macOS 专用: `allowUnixSockets`, `enableWeakerNetworkIsolation`
   - Linux 忽略 Unix socket 路径 (seccomp 无法按路径过滤)

4. **passthrough 模式**:
   - 允许未记录的设置通过
   - `enabledPlatforms` 未记录但可通过

## 设计要点

1. **单一真相源**:
   - SDK 和设置验证都从此文件导入
   - 确保类型一致性

2. **Zod v4**:
   - 使用 zod/v4 进行 schema 定义
   - 支持高级验证特性

3. **描述性文档**:
   - 每个选项都有详细描述
   - 安全影响明确说明

4. **灵活性**:
   - 支持策略设置覆盖
   - 用户、项目、本地设置分层

## 与其他文件的关系

- **../utils/lazySchema.js**: lazySchema 实现
- **SDK**: 导入 sandbox 类型
- **Settings validation**: 验证沙箱配置

## 注意事项

1. `allowManagedDomainsOnly` 需要托管设置才能生效
2. `enabledPlatforms` 是未记录的设置
3. `enableWeakerNetworkIsolation` 有安全风险
4. `failIfUnavailable` 用于强制沙箱的托管部署
5. `ignoreViolations` 用于临时绕过违规检查
6. 路径权限与 Edit/Read 工具的权限规则合并
