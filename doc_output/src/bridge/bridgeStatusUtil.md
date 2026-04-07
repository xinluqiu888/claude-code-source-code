# bridgeStatusUtil.ts — 桥接状态管理工具函数

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/bridge/bridgeStatusUtil.ts`
- **文件类型**: TypeScript 模块
- **行数**: 约 164 行
- **主要职责**: 提供桥接状态相关的工具函数，包括状态机、时间格式化、URL 构建、微光动画计算和终端超链接

## 功能概述

`bridgeStatusUtil.ts` 是桥接 UI 层的状态管理工具模块，负责处理连接状态的表示、格式化辅助功能以及终端显示效果。该模块独立于具体的渲染实现（chalk 或 React/Ink），提供纯计算函数，可被 CLI 桥接和 TUI 桥接共用。

模块涵盖以下主要功能领域：状态机类型定义和状态派生；时间格式化和文本截断；桥接连接 URL 和会话 URL 构建；微光动画（shimmer）的视觉效果计算；终端 OSC 8 超链接包装；以及状态标签和颜色的确定。

## 核心内容详解

### 类型定义

#### `StatusState` 类型
桥接状态机状态：
- `'idle'` - 空闲/就绪状态
- `'attached'` - 已附加到会话
- `'titled'` - 会话已命名
- `'reconnecting'` - 正在重新连接
- `'failed'` - 连接失败

#### `BridgeStatusInfo` 类型
状态信息包含：
- `label`: 状态标签文本（'Remote Control failed'、'Remote Control reconnecting'、'Remote Control active'、'Remote Control connecting…'）
- `color`: 状态颜色（'error'、'warning'、'success'）

### 常量定义

| 常量 | 值 | 说明 |
|------|-----|------|
| `TOOL_DISPLAY_EXPIRY_MS` | 30_000 | 工具活动行在最后一次 tool_start 后保持可见的时间 |
| `SHIMMER_INTERVAL_MS` | 150 | 微光动画刻度间隔 |

### 工具函数

#### `timestamp(): string`
生成当前时间的 HH:MM:SS 格式时间戳。使用本地时间，数字补零到两位。

#### `formatDuration` / `truncatePrompt`
从 `../utils/format.js` 重新导出的函数，分别用于格式化持续时间和截断提示文本。

#### `abbreviateActivity(summary: string): string`
缩写工具活动摘要，截断到 30 个字符宽度。用于状态行的第二行显示。

#### `buildBridgeConnectUrl(environmentId, ingressUrl?): string`
构建桥接连接 URL（空闲状态显示）：
- 基础 URL: `getClaudeAiBaseUrl(undefined, ingressUrl)`
- 格式: `{baseUrl}/code?bridge={environmentId}`

#### `buildBridgeSessionUrl(sessionId, environmentId, ingressUrl?): string`
构建桥接会话 URL（会话附加后显示）：
- 委托给 `getRemoteSessionUrl` 进行会话 ID 前缀转换（`cse_` → `session_`）
- 追加 `?bridge={environmentId}` 查询参数

#### `computeGlimmerIndex(tick, messageWidth): number`
计算反向扫过微光动画的微光索引：
- 周期长度: `messageWidth + 20`
- 返回: `messageWidth + 10 - (tick % cycleLength)`
- 产生从右向左扫过的微光效果

#### `computeShimmerSegments(text, glimmerIndex): { before, shimmer, after }`
将文本按视觉列位置分割为三段，用于微光渲染：

**算法**:
1. 计算消息总宽度（使用 `stringWidth`）
2. 计算微光起始和结束位置（`glimmerIndex - 1` 到 `glimmerIndex + 1`）
3. 如果微光完全在屏幕外，返回全部作为 `before`
4. 使用 grapheme 分词器逐字符处理
5. 根据列位置将字符分配到 `before`、`shimmer` 或 `after`

**支持特性**:
- 多字节字符（emoji、CJK）
- Grapheme 边界感知（使用 `getGraphemeSegmenter`）
- 视觉宽度计算（使用 `stringWidth`）

#### `getBridgeStatus({ error, connected, sessionActive, reconnecting }): BridgeStatusInfo`
从连接状态派生状态标签和颜色：

| 条件 | 标签 | 颜色 |
|------|------|------|
| `error` 存在 | 'Remote Control failed' | error |
| `reconnecting` | 'Remote Control reconnecting' | warning |
| `sessionActive \|\| connected` | 'Remote Control active' | success |
| 默认 | 'Remote Control connecting…' | warning |

#### `buildIdleFooterText(url): string`
空闲状态的页脚文本：`'Code everywhere with the Claude app or {url}'`

#### `buildActiveFooterText(url): string`
活动状态的页脚文本：`'Continue coding in the Claude app or {url}'`

#### `FAILED_FOOTER_TEXT`
失败状态页脚常量：`'Something went wrong, please try again'`

#### `wrapWithOsc8Link(text, url): string`
将文本包装在 OSC 8 终端超链接序列中：
- 格式: `\x1b]8;;{url}\x07{text}\x1b]8;;\x07`
- 视觉宽度为零（被 `stringWidth` 正确剥离）
- 支持在支持 OSC 8 的终端中可点击

## 设计要点

1. **渲染无关**: 所有函数返回原始数据和字符串，由调用者（`bridgeUI.ts` 使用 chalk，`bridge.tsx` 使用 React/Ink）应用样式。

2. **国际化支持**: 使用 grapheme 分词器正确处理 Unicode，包括 emoji 和 CJK 字符的组合序列。

3. **视觉宽度感知**: 使用 `stringWidth` 而非字符串长度，正确处理全角字符和零宽度字符。

4. **微光动画数学**: 反向扫过效果通过计算周期内的递减位置实现，产生从右向左的扫过感。

5. **状态机完整**: 覆盖所有可能的状态转换，没有未处理的分支。

6. **URL 构建统一**: 环境 ID 和入口 URL 的处理集中在此模块，确保 URL 格式一致性。

## 与其他文件的关系

| 文件 | 关系描述 |
|------|----------|
| `../constants/product.js` | 导入 `getClaudeAiBaseUrl` 和 `getRemoteSessionUrl` |
| `../ink/stringWidth.js` | 导入 `stringWidth` 计算视觉宽度 |
| `../utils/format.js` | 导入 `formatDuration` 和 `truncateToWidth` |
| `../utils/intl.js` | 导入 `getGraphemeSegmenter` 用于 grapheme 分割 |
| `bridgeUI.ts` | 调用本模块函数进行状态渲染 |
| `bridge.tsx` | 调用 `computeShimmerSegments` 和 `getBridgeStatus` 进行 TUI 渲染 |

## 注意事项

1. **时间戳本地化**: `timestamp()` 使用本地时区，不涉及国际化处理。

2. **微光宽度**: `computeShimmerSegments` 产生的微光区域宽度为 3 个字符（`glimmerIndex - 1` 到 `glimmerIndex + 1`），但受 grapheme 边界对齐影响可能略有差异。

3. **OSC 8 兼容性**: `wrapWithOsc8Link` 生成的序列在老式终端中会被静默忽略，显示为普通文本。`stringWidth` 正确剥离这些序列，确保布局计算准确。

4. **URL 硬编码**: `buildIdleFooterText` 和 `buildActiveFooterText` 中的英文文本是硬编码的，没有国际化支持。

5. **状态优先级**: `getBridgeStatus` 中的条件顺序很重要——`error` 优先于 `reconnecting`，`reconnecting` 优先于 `connected`。

6. **性能考虑**: `computeShimmerSegments` 每次调用都重新分词，对于高频动画（150ms 间隔）可能需要注意性能，但通常消息长度有限，影响不大。
