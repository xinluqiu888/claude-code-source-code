# useIDEStatusIndicator.tsx — IDE 状态指示器

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/hooks/notifs/useIDEStatusIndicator.tsx`
- **类型**: React Hook (TSX)
- **导出函数**: `useIDEStatusIndicator`
- **依赖**: React useEffect/useRef/useMemo, notifications, IDE utils

## 功能概述

本 Hook 管理 IDE 连接状态的各种通知，包括：
1. IDE 连接断开提示
2. IDE 安装错误提示
3. JetBrains 插件未连接提示
4. /ide 命令提示（外部终端运行时）

## 核心内容详解

### 参数

```typescript
type Props = {
  ideInstallationStatus: IDEExtensionInstallationStatus | null
  ideSelection: IDESelection | undefined
  mcpClients: MCPServerConnection[]
}
```

### 状态计算

```typescript
const isJetBrains = ideInstallationStatus 
  ? isJetBrainsIde(ideInstallationStatus?.ideType) 
  : false

const showIDEInstallErrorOrJetBrainsInfo = 
  ideInstallationStatus?.error || isJetBrains

const shouldShowIdeSelection = 
  ideStatus === 'connected' && 
  (ideSelection?.filePath || (ideSelection?.text && ideSelection.lineCount > 0))

const shouldShowConnected = 
  ideStatus === 'connected' && !shouldShowIdeSelection

const showIDEInstallError = 
  showIDEInstallErrorOrJetBrainsInfo && !isJetBrains && 
  !shouldShowConnected && !shouldShowIdeSelection

const showJetBrainsInfo = 
  showIDEInstallErrorOrJetBrainsInfo && isJetBrains && 
  !shouldShowConnected && !shouldShowIdeSelection
```

### 通知类型

**1. /ide 命令提示**
- 条件：非支持终端、无 IDE 连接、非 JetBrains 信息模式、提示未显示过、未超过最大次数
- 延迟 3 秒显示，避免自动连接期间的闪烁

**2. IDE 断开连接**
- 条件：IDE 状态为 'disconnected' 且有 IDE 名称
- 显示："{ideName} disconnected"

**3. JetBrains 未连接**
- 条件：需要显示 JetBrains 信息
- 显示："IDE plugin not connected · /status for info"

**4. 安装错误**
- 条件：需要显示安装错误
- 显示："IDE extension install failed (see /status for info)"

### 显示限制

```typescript
const MAX_IDE_HINT_SHOW_COUNT = 5
```

/ide 提示最多显示 5 次。

## 设计要点

### 1. 状态推导

通过多个布尔变量组合决定显示哪种通知，避免复杂嵌套。

### 2. 延迟提示

3 秒延迟避免自动连接期间的提示闪烁。

### 3. 次数限制

使用全局配置存储提示显示次数。

### 4. IDE 检测

异步检测运行的 IDE，动态显示名称。

### 5. 互斥通知

不同通知之间有互斥关系，避免同时显示多个冲突通知。

## 与其他文件的关系

- **useIdeConnectionStatus.ts**: 获取 IDE 连接状态
- **ide.ts**: IDE 工具函数
- **config.ts**: 全局配置读写

## 注意事项

1. **多种 IDE 支持**: VS Code、JetBrains 等不同 IDE 有不同处理逻辑
2. **外部终端检测**: 仅在非内置终端显示 /ide 提示
3. **自动连接**: 需要考虑自动连接过程中的状态变化
4. **错误类型**: 区分安装错误和连接错误
