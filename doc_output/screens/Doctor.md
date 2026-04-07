# Doctor.tsx — 诊断信息屏幕组件

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/screens/Doctor.tsx`
- **类型**: React TSX 组件
- **作用**: 显示Claude Code的安装和运行状态诊断信息

## 功能概述

Doctor组件提供了一个综合的诊断界面，用于检查和展示Claude Code的各个方面状态，包括安装类型、版本信息、Agent配置、MCP工具、设置错误、环境变量验证等。用户可以通过`/doctor`命令访问此界面。

## 核心内容详解

### 组件Props

```typescript
type Props = {
  onDone: (result?: string, options?: { display?: CommandResultDisplay }) => void
}
```

### 内部状态类型

```typescript
type AgentInfo = {
  activeAgents: Array<{ agentType: string; source: SettingSource | 'built-in' | 'plugin' }>
  userAgentsDir: string
  projectAgentsDir: string
  userDirExists: boolean
  projectDirExists: boolean
  failedFiles?: Array<{ path: string; error: string }>
}

type VersionLockInfo = {
  enabled: boolean
  locks: LockInfo[]
  locksDir: string
  staleLocksCleaned: number
}
```

### 诊断数据

通过`getDoctorDiagnostic()`获取的核心诊断信息：
- **installationType**: 安装类型（npm全局、本地开发等）
- **version**: 当前版本
- **packageManager**: 使用的包管理器
- **installationPath**: 安装路径
- **invokedBinary**: 调用的二进制文件

### 子组件

#### `DistTagsDisplay`
显示版本分布标签：
- Stable版本（如果存在）
- Latest版本
- 获取失败提示

### 检查项

1. **Agent配置检查**:
   - 活动Agent列表及来源
   - 用户级Agent目录（~/.claude/agents）
   - 项目级Agent目录（./.claude/agents）
   - 加载失败的文件

2. **MCP工具检查**:
   - 已连接的MCP服务器
   - 工具权限上下文

3. **设置验证错误**:
   - 通过`useSettingsErrors()`获取
   - 排除MCP相关错误

4. **环境变量验证**:
   - `BASH_MAX_OUTPUT_LENGTH`（默认/上限: 1MB/50MB）
   - `TASK_MAX_OUTPUT_LENGTH`（默认/上限: 1MB/50MB）
   - `CLAUDE_CODE_MAX_OUTPUT_TOKENS`

5. **版本锁定信息**（PID-based）:
   - 是否启用
   - 锁定目录位置
   - 已清理的过时锁定数
   - 当前所有锁定信息

6. **上下文警告**:
   - 通过`checkContextWarnings()`检查
   - 包括Agent上下文警告

### 用户界面

- **标题**: "Diagnostics"
- **加载状态**: "Checking installation status…"
- **分类显示**: 安装信息、版本信息、Agent、MCP、错误、环境变量、版本锁定等
- **确认关闭**: 按Enter或确认键关闭

### 按键绑定

```typescript
{
  "confirm:yes": handleDismiss,
  "confirm:no": handleDismiss
}
```

## 设计要点

1. **React Compiler优化**: 使用React Compiler运行时（`c as _c`）进行自动记忆化
2. **异步数据加载**: 使用useEffect并行加载多个诊断数据源
3. **渐进式显示**: 先显示基本诊断，Agent信息等异步加载后更新
4. **错误处理**: 非ENOENT错误会记录到调试日志

## 与其他文件的关系

- **使用 doctorDiagnostic.ts**: `getDoctorDiagnostic()`获取核心诊断
- **使用 doctorContextWarnings.ts**: `checkContextWarnings()`检查上下文警告
- **使用 useSettingsErrors hook**: 获取设置验证错误
- **使用 SandboxDoctorSection**: 显示沙箱相关信息
- **使用 KeybindingWarnings**: 显示按键绑定警告
- **使用 McpParsingWarnings**: 显示MCP解析警告
- **使用 ValidationErrorsList**: 显示验证错误列表

## 注意事项

1. **PID锁定**: 仅当`isPidBasedLockingEnabled()`返回true时显示版本锁定信息
2. **自动更新通道**: 显示当前配置的自动更新通道（stable/latest）
3. **退出处理**: 使用`useExitOnCtrlCDWithKeybindings`处理Ctrl-C退出
4. **Agent失败文件**: 记录加载失败的Agent文件路径和错误信息
