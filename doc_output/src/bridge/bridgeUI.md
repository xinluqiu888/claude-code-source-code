# bridgeUI.ts — 桥接命令行界面渲染

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/bridge/bridgeUI.ts`
- **文件类型**: TypeScript 模块
- **行数**: 约 531 行
- **主要职责**: 提供命令行环境下的桥接状态渲染，包括 QR 码生成、状态行管理和日志输出

## 功能概述

`bridgeUI.ts` 实现了一个基于 chalk 的命令行桥接状态显示器。它负责渲染桥接启动横幅、连接状态、会话信息、工具活动以及 QR 码。该模块专为非 React CLI 环境设计，与 TUI 版本的 `bridge.tsx` 形成互补。

模块的核心是 `createBridgeLogger` 工厂函数，返回一个包含多种渲染方法的对象。该设计将状态管理与渲染逻辑封装在一起，通过闭包维护内部状态（连接状态、会话计数、工具活动等）。

主要功能包括：状态行的动态更新和清除；连接微光动画；QR 码生成和显示切换；多会话模式下的会话列表渲染；工具活动摘要的短暂显示；以及错误和重新连接状态的视觉反馈。

## 核心内容详解

### QR 码选项

```typescript
const QR_OPTIONS = {
  type: 'utf8',
  errorCorrectionLevel: 'L',  // 低纠错级别，更小尺寸
  small: true,                // 紧凑模式
}
```

### createBridgeLogger 工厂函数

**参数**:
- `verbose: boolean` - 详细模式
- `write?: (s: string) => void` - 自定义输出函数（默认为 `process.stdout.write`）

**返回**: `BridgeLogger` 对象

### 内部状态

| 状态变量 | 说明 |
|----------|------|
| `statusLineCount` | 当前显示的状态行数（用于清除） |
| `currentState` / `currentStateText` | 当前状态机和显示文本 |
| `repoName` / `branch` | 仓库名和分支名 |
| `connectUrl` / `activeSessionUrl` | 连接 URL 和活动会话 URL |
| `qrLines` / `qrVisible` | QR 码行数据和显示开关 |
| `lastToolSummary` / `lastToolTime` | 最后工具活动和显示时间 |
| `sessionActive` / `sessionMax` | 活跃会话数和最大会话数 |
| `spawnMode` / `spawnModeDisplay` | 生成模式和显示模式 |
| `sessionDisplayInfo` | 会话显示信息映射（ID → {title, url, activity}） |
| `connectingTimer` / `connectingTick` | 连接动画定时器和帧索引 |

### 核心方法

#### `countVisualLines(text): number`
计算字符串在终端中的视觉行数，考虑：
- 换行符分割逻辑行
- 终端列宽（`process.stdout.columns` 或 80 作为回退）
- 字符串视觉宽度（使用 `stringWidth`）
- 空段处理

#### `writeStatus(text)` / `clearStatusLines()`
状态行写入和清除：
- `writeStatus`: 写入文本并增加 `statusLineCount`
- `clearStatusLines`: 使用 ANSI 序列上移并清除 `statusLineCount` 行

#### `printLog(line)`
打印永久日志行：先清除状态行，写入日志，然后重新渲染状态。

#### `renderConnectingLine()` / `startConnecting()` / `stopConnecting()`
连接动画管理：
- 使用 `BRIDGE_SPINNER_FRAMES` 帧数组
- 150ms 间隔的循环动画
- 显示 "Connecting" 文本和仓库/分支后缀

#### `renderStatusLine()`
核心状态渲染函数：
1. 检查状态（非 reconnecting/failed 才渲染）
2. 清除现有状态行
3. 如启用 QR 码，先渲染 QR 码
4. 根据状态渲染状态指示器和文本
5. 多会话模式：渲染容量提示和会话列表
6. 单会话模式：渲染生成模式提示
7. 活动工具摘要（未过期时）
8. 页脚 URL 和操作提示

#### 公开方法

| 方法 | 说明 |
|------|------|
| `printBanner(config, environmentId)` | 打印启动横幅，启动连接动画，初始化 QR 码 |
| `logSessionStart(sessionId, prompt)` | 记录会话开始（详细模式） |
| `logSessionComplete(sessionId, durationMs)` | 记录会话完成 |
| `logSessionFailed(sessionId, error)` | 记录会话失败 |
| `logStatus(message)` / `logVerbose(message)` | 状态日志（后者受 verbose 控制） |
| `logError(message)` | 错误日志（红色） |
| `logReconnected(disconnectedMs)` | 重新连接日志 |
| `setRepoInfo(repo, branch)` | 设置仓库信息 |
| `setDebugLogPath(path)` | 设置调试日志路径 |
| `updateIdleStatus()` | 更新为空闲状态，停止动画，重置 QR 码 |
| `setAttached(sessionId)` | 设置为已附加状态，更新 URL |
| `updateReconnectingStatus(delayStr, elapsedStr)` | 渲染重新连接状态（带旋转器） |
| `updateFailedStatus(error)` | 渲染失败状态和错误信息 |
| `updateSessionStatus(...)` | 更新会话状态和工具活动 |
| `clearStatus()` | 清除所有状态显示 |
| `toggleQr()` | 切换 QR 码显示 |
| `updateSessionCount(active, max, mode)` | 更新会话计数 |
| `setSpawnModeDisplay(mode)` | 设置生成模式显示 |
| `addSession(sessionId, url)` | 添加会话到列表 |
| `updateSessionActivity(sessionId, activity)` | 更新会话活动 |
| `setSessionTitle(sessionId, title)` | 设置会话标题 |
| `removeSession(sessionId)` | 从列表移除会话 |
| `refreshDisplay()` | 刷新显示（重新渲染） |

### 视觉样式

使用 chalk 应用颜色：
- 绿色: 就绪/成功状态
- 青色: 已连接状态
- 黄色: 连接中/警告
- 红色: 失败/错误
- 暗淡（dim）: 辅助信息

## 设计要点

1. **ANSI 序列管理**: 精确跟踪状态行数量，使用 `\x1b[N A` 上移和 `\x1b[J` 清除，避免屏幕残留。

2. **状态行生命周期**: 日志输出必须清除状态行 → 写日志 → 重新渲染状态，确保状态行始终在底部。

3. **QR 码延迟生成**: QR 码生成是异步的，先生成默认内容，完成后再更新。

4. **动画帧共享**: 使用 `BRIDGE_SPINNER_FRAMES` 常量，确保与 TUI 版本动画一致。

5. **多会话列表**: 使用 `Map` 存储会话信息，按添加顺序迭代显示。

6. **URL 超链接**: 使用 `wrapWithOsc8Link` 生成可点击的终端超链接。

7. **工具活动过期**: `TOOL_DISPLAY_EXPIRY_MS` 后自动隐藏工具活动摘要。

8. **工作树分支隐藏**: 在 worktree 模式下不显示分支名（每个会话有自己的分支）。

## 与其他文件的关系

| 文件 | 关系描述 |
|------|----------|
| `../constants/figures.js` | 导入指示器符号和旋转器帧 |
| `../ink/stringWidth.js` | 视觉宽度计算 |
| `./bridgeStatusUtil.js` | 导入状态工具函数（时间戳、URL 构建、截断等） |
| `./types.js` | 导入 `BridgeConfig`, `BridgeLogger`, `SessionActivity`, `SpawnMode` 类型 |
| `bridgeMain.ts` | 创建和使用 logger 实例 |

## 注意事项

1. **终端宽度回退**: 当 `process.stdout.columns` 不可用时使用 80 作为回退，可能导致宽终端中的行计算不准确。

2. **ANT 用户日志路径**: 当 `USER_TYPE === 'ant'` 时显示调试日志路径，这是内部用户的诊断功能。

3. **生成模式提示**: 空格键切换 QR 码，'w' 键切换生成模式（仅在 `spawnModeDisplay` 设置时）。

4. **QR 码 URL 策略**: 
   - 单会话模式：附加会话后 QR 码指向会话 URL
   - 多会话模式：QR 码始终指向环境连接 URL（以便生成更多会话）

5. **会话标题截断**: 多会话列表中标题截断到 35 字符，单会话状态行截断到 40 字符。

6. **状态保护**: `reconnecting` 和 `failed` 状态有特殊处理——某些操作（如 `setSessionTitle`）在这些状态下不重新渲染，以免破坏错误显示。

7. **字符串宽度计算**: 使用 `stringWidth` 而非 `length`，正确处理 emoji 和 CJK 字符的视觉宽度。
