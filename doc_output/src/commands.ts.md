# commands.ts — 命令注册与管理中心

> **一句话总结**：Claude Code 所有斜杠命令的注册中心，负责加载、过滤和管理内置命令、技能、插件及工作流命令。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~755 行 |
| 主要职责 | 注册和管理所有可用的斜杠命令，包括内置命令、技能、插件和工作流 |

---

## 功能概述

`commands.ts` 是 Claude Code 的命令注册中心，负责收集、组织和提供所有可用的斜杠命令（如 `/help`、`/commit`、`/skills` 等）。该文件通过动态导入和条件编译支持多种功能特性（feature flags），使得不同构建版本可以包含不同的命令集。

该模块采用记忆化（memoization）技术优化性能，避免重复加载命令。同时，它还处理命令的可用性检查（基于用户身份验证状态和服务提供商），并支持动态技能的运行时发现。

---

## 核心内容详解

### 导入与依赖

**主要依赖模块：**
- `lodash-es/memoize.js` - 用于记忆化命令加载函数
- `bun:bundle` 的 `feature` 函数 - 条件编译特性检测
- 各类命令实现文件（位于 `src/commands/` 目录下）
- 技能、插件相关工具函数

### 主要类型与接口

- **Command**: 命令的基础类型定义（从 `./types/command.js` 导入）
- **CommandResultDisplay**: 命令结果显示类型
- **PromptCommand**: 提示类型命令接口

### 核心函数与常量

#### `COMMANDS` (记忆化函数)
返回所有内置命令的列表。使用 `memoize` 包装，确保多次调用返回相同结果。

```typescript
const COMMANDS = memoize((): Command[] => [
  addDir, advisor, agents, branch, btw, chrome, clear, ...
])
```

#### `INTERNAL_ONLY_COMMANDS`
仅限内部使用的命令列表（如 `backfillSessions`、`breakCache` 等），仅在 `USER_TYPE === 'ant'` 时可用。

#### `REMOTE_SAFE_COMMANDS`
远程模式（`--remote`）下安全的命令集合。这些命令只影响本地 TUI 状态，不依赖本地文件系统、git、shell、IDE 等。

包含的命令：
- `session` - 显示远程会话二维码/URL
- `exit` - 退出 TUI
- `clear` - 清屏
- `help` - 显示帮助
- `theme`/`color` - 更改主题/颜色
- `cost`/`usage` - 显示成本/使用信息
- 等等

#### `BRIDGE_SAFE_COMMANDS`
通过 Remote Control Bridge 执行安全的本地命令集合。这些命令产生可流式传输到移动/网页客户端的文本输出。

### 主要导出函数

#### `getCommands(cwd: string): Promise<Command[]>`
返回当前用户可用的所有命令。

**执行流程：**
1. 调用 `loadAllCommands(cwd)` 加载所有命令源
2. 获取动态发现的技能
3. 过滤掉不满足可用性要求的命令
4. 将动态技能插入到内置命令之前

#### `loadAllCommands(cwd: string): Promise<Command[]>`
加载所有命令源（技能、插件、工作流），按特定顺序组合：
1. 捆绑技能（bundled skills）
2. 内置插件技能
3. 技能目录命令
4. 工作流命令
5. 插件命令
6. 插件技能
7. 内置命令

#### `getSkillToolCommands(cwd: string): Promise<Command[]>`
返回模型可调用的所有提示类型命令（prompt-type commands）。

**过滤条件：**
- 类型为 `'prompt'`
- 不禁用模型调用
- 来源不是 `'builtin'`
- 满足特定来源条件（bundled、skills、commands_DEPRECATED 或有用户指定描述）

#### `getSlashCommandToolSkills(cwd: string): Promise<Command[]>`
返回所有技能命令，用于斜杠命令工具。

#### `meetsAvailabilityRequirement(cmd: Command): boolean`
检查命令是否满足其声明的可用性要求。

支持的可用性类型：
- `'claude-ai'` - 需要 Claude.ai 订阅用户
- `'console'` - 控制台 API 密钥用户（直接 Anthropic API 客户）

#### `clearCommandsCache(): void`
清除所有命令相关的缓存，用于在动态技能添加时使缓存失效。

#### `findCommand(commandName: string, commands: Command[]): Command | undefined`
按名称或别名查找命令。

#### `formatDescriptionWithSource(cmd: Command): string`
格式化命令描述，附加来源标注（如 `(plugin)`、`(bundled)` 等）。

---

## 设计要点

### 条件编译（Dead Code Elimination）
大量使用 `feature('XXX')` 进行条件编译，未启用的功能在构建时会被消除，减小包体积：

```typescript
const voiceCommand = feature('VOICE_MODE')
  ? require('./commands/voice/index.js').default
  : null
```

### 记忆化缓存策略
- `COMMANDS`、`loadAllCommands`、`getSkillToolCommands` 等均使用 `memoize` 包装
- `getCommands` 不缓存，因为认证状态可能变化
- 提供 `clearCommandMemoizationCaches()` 用于选择性清除缓存

### 命令来源分层
命令按来源优先级排序，确保用户自定义内容优先于内置内容：
1. 捆绑技能（最高优先级）
2. 内置插件技能
3. 用户技能目录
4. 工作流脚本
5. 插件命令/技能
6. 内置命令（最低优先级）

---

## 与其他文件的关系

### 依赖
- `src/types/command.ts` - 命令类型定义
- `src/commands/*/index.ts` - 各命令实现
- `src/skills/loadSkillsDir.ts` - 技能加载
- `src/plugins/builtinPlugins.ts` - 内置插件
- `src/utils/plugins/loadPluginCommands.ts` - 插件命令加载
- `src/utils/auth.ts` - 认证状态检查

### 被依赖
- `src/main.tsx` - 主入口，用于命令预过滤
- `src/QueryEngine.ts` - 查询引擎，获取可用命令
- `src/tools/SkillTool.ts` - 技能工具，获取可调用的技能命令
- `src/components/HelpV2/HelpV2.tsx` - 帮助界面

---

## 注意事项

1. **模块初始化限制**: 底层函数在模块初始化时不能读取配置，因此使用函数包装延迟执行
2. **动态技能去重**: 添加动态技能时会检查是否已存在同名命令，避免重复
3. **远程模式限制**: 远程模式下只有 `REMOTE_SAFE_COMMANDS` 中的命令可用
4. **Bridge 安全**: 通过 Bridge 接收的命令需要额外安全检查（`isBridgeSafeCommand`）
