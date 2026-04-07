# ShellSnapshot.ts — Shell环境快照管理器

> **一句话总结**：捕获用户Shell配置（函数、别名、选项）并创建可复用的快照文件，确保命令在用户预期的环境中执行。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/ShellSnapshot.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~583行 |
| 主要职责 | 创建和管理Shell环境快照，支持Bash和Zsh |

---

## 功能概述

ShellSnapshot模块是Claude Code实现"用户Shell感知"的核心组件。由于Claude Code使用非交互式子进程执行命令，它无法直接继承用户的Shell配置（如.bashrc或.zshrc中定义的函数、别名和环境变量）。

该模块通过以下方式解决此问题：
1. 启动用户指定的Shell并加载其配置文件
2. 捕获所有用户定义的函数、别名和Shell选项
3. 将这些内容写入临时快照文件
4. 后续命令执行时先source该快照文件

此外，该模块还集成了嵌入式搜索工具（ripgrep、bfs、ugrep），在用户环境中自动创建对应的别名或函数。

---

## 核心内容详解

### 主要导出函数

- **`createAndSaveSnapshot`**
  - 参数：`binShell`（string）- Shell二进制路径
  - 返回值：`Promise<string | undefined>` - 快照文件路径或undefined
  - 用途：主入口函数，创建并保存Shell环境快照
  - 超时：10秒（SNAPSHOT_CREATION_TIMEOUT）

### 嵌入式搜索工具集成

- **`createRipgrepShellIntegration`**
  - 返回值：对象包含`type`（'alias' | 'function'）和`snippet`
  - 用途：创建ripgrep的Shell集成
  - 特殊处理：对于嵌入式ripgrep，使用ARGV0技巧调用bun内置工具

- **`createFindGrepShellIntegration`**
  - 返回值：字符串或null
  - 用途：创建find/grep的替代实现（使用bfs/ugrep）
  - 特性：仅在ant-native构建中可用，始终覆盖系统find/grep

### 快照内容生成

- **`getUserSnapshotContent`**
  - 参数：`configFile`（string）- 配置文件路径
  - 用途：生成用户特定的快照内容（函数、选项、别名）
  - 差异处理：Bash和Zsh有不同的命令语法

- **`getClaudeCodeSnapshotContent`**
  - 用途：生成Claude Code特定的快照内容
  - 包含：PATH设置、ripgrep集成、find/grep集成

---

## 设计要点

1. **跨Shell支持**：分别处理Bash和Zsh的语法差异（如`declare -f` vs `typeset -f`）
2. **ARGV0技巧**：使用Bun内置的ARGV0调度机制调用嵌入式搜索工具
3. **Windows兼容**：处理Git Bash环境下的特殊问题（如winpty别名过滤）
4. **安全清理**：注册cleanup函数在会话结束时删除快照文件
5. **错误处理**：详细记录快照创建失败的各种诊断信息

---

## 与其他文件的关系

- **依赖**：
  - `../cwd.ts` - 获取当前工作目录
  - `../envUtils.ts` - 获取Claude配置目录
  - `./shellQuote.ts` - 命令引用工具
  - `../ripgrep.ts` - ripgrep命令配置
- **被依赖**：
  - `../Shell.ts` - 在Shell执行前加载快照
  - `../sessionStart.ts` - 会话初始化时创建快照

---

## 注意事项

- 快照创建失败不会阻止Claude Code运行，只是降级到无用户配置模式
- 嵌入式搜索工具仅在特定构建（ant-native）中可用
- 对于Windows Git Bash，会过滤掉可能导致问题的winpty别名
- 快照文件存储在`~/.config/claude/shell-snapshots/`目录下
