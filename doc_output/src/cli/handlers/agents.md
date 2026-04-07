# handlers/agents.ts — Agents 子命令处理器

## 基本信息

| 属性 | 值 |
|------|-----|
| **文件路径** | `/root/projects/claude-code-source-code/src/cli/handlers/agents.ts` |
| **文件类型** | TypeScript 处理器模块 |
| **行数** | 71 行 |
| **职责** | 实现 `claude agents` 命令，列出当前配置的代理列表 |

## 功能概述

本模块实现 `claude agents` 子命令，用于显示当前工作目录下配置的所有代理（Agents）。它动态加载代理定义，解析覆盖关系，并按来源分组显示。

代理是 Claude Code 的可配置 AI 助手角色，每个代理可以有不同的系统提示、模型、内存等配置。本处理器帮助用户了解当前可用的代理及其来源。

## 核心内容详解

### 导入

| 导入项 | 来源 | 用途 |
|--------|------|------|
| `AGENT_SOURCE_GROUPS`, `compareAgentsByName`, `getOverrideSourceLabel`, `ResolvedAgent`, `resolveAgentModelDisplay`, `resolveAgentOverrides` | `../../tools/AgentTool/agentDisplay.js` | 代理显示工具函数 |
| `getActiveAgentsFromList`, `getAgentDefinitionsWithOverrides` | `../../tools/AgentTool/loadAgentsDir.js` | 代理定义加载 |
| `getCwd` | `../../utils/cwd.js` | 获取当前工作目录 |

### 辅助函数

#### `formatAgent(agent: ResolvedAgent): string`
格式化单个代理为可读的字符串表示。

**参数**：
- `agent`: 已解析的代理定义

**返回值格式**：
```
agentType · model · memory memory
```

**示例**：
- `code · claude-3-opus-20240229`
- `reviewer · claude-3-sonnet-20240229 · 500K memory`

### 主函数

#### `export async function agentsHandler(): Promise<void>`

agents 子命令的主处理函数。

**执行流程**：

1. **获取当前工作目录**
   ```typescript
   const cwd = getCwd()
   ```

2. **加载代理定义**
   ```typescript
   const { allAgents } = await getAgentDefinitionsWithOverrides(cwd)
   ```
   - 从磁盘加载所有代理定义
   - 包括内置代理、项目代理、用户代理等

3. **获取活跃代理**
   ```typescript
   const activeAgents = getActiveAgentsFromList(allAgents)
   ```
   - 过滤出当前活跃的代理

4. **解析覆盖关系**
   ```typescript
   const resolvedAgents = resolveAgentOverrides(allAgents, activeAgents)
   ```
   - 确定哪些代理被其他同名代理覆盖
   - 标记被覆盖的代理和覆盖来源

5. **按来源分组显示**
   - 遍历 `AGENT_SOURCE_GROUPS` 定义的来源组
   - 对每个组：
     - 过滤出该来源的代理
     - 按名称排序
     - 添加组标题
     - 格式化每个代理：
       - 如果被覆盖，显示 `(shadowed by {source})`
       - 否则显示代理信息

6. **输出结果**
   - 如果没有代理，显示 "No agents found."
   - 否则显示活跃代理数量和详细列表

### 代理来源组 (AGENT_SOURCE_GROUPS)

| 标签 | 来源 | 描述 |
|------|------|------|
| Built-in | `builtIn` | 内置代理 |
| Project | `project` | 项目级配置代理（.claude/agents/）|
| Local | `local` | 本地用户代理 |
| Global | `global` | 全局用户代理 |

## 设计要点

1. **动态加载**：仅在命令执行时加载代理定义，确保显示最新配置
2. **来源感知**：清晰显示每个代理的来源，帮助用户理解配置层级
3. **覆盖透明**：明确标记被覆盖的代理及其覆盖来源，避免混淆
4. **排序一致**：按名称排序，提供一致的输出
5. **简洁格式**：使用 `·` 分隔符提供简洁的代理信息概览
6. **懒加载**：模块设计为动态导入，仅在执行 `claude agents` 时加载

## 与其他文件的关系

| 关系类型 | 文件 | 描述 |
|---------|------|------|
| **导入** | `src/tools/AgentTool/agentDisplay.js` | 代理显示工具函数 |
| **导入** | `src/tools/AgentTool/loadAgentsDir.js` | 代理定义加载函数 |
| **导入** | `src/utils/cwd.js` | 获取当前工作目录 |
| **被导入** | `src/main.tsx` | `claude agents` 命令处理（动态导入）|

## 注意事项

1. **动态导入**：模块注释说明它是动态导入的，仅在 `claude agents` 运行时加载
2. **输出到 console.log**：使用 `console.log` 进行输出，符合 CLI 命令的行为模式
3. **空行处理**：在组之间添加空行，在末尾使用 `trimEnd()` 清理
4. **活跃代理计数**：仅统计未被覆盖的代理作为 "active" 数量
5. **异步操作**：使用 `async/await` 处理代理定义的异步加载
