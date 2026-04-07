# shellCompletion.ts — Shell命令补全服务

> **一句话总结**：基于用户系统的Bash/Zsh Shell提供实时的命令、变量和文件名补全功能。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/shellCompletion.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~260行 |
| 主要职责 | 集成Bash/Zsh的compgen功能提供命令补全 |

---

## 功能概述

该模块为Claude Code的输入提示提供Shell级别的自动补全。它通过：
1. 解析当前输入确定补全上下文（命令、变量、文件）
2. 生成对应的Shell命令（compgen for bash, 原生zsh命令）
3. 执行Shell命令获取补全候选
4. 返回格式化的SuggestionItem数组

支持的补全类型：
- **命令补全**：输入位置期望命令时（行首或操作符后）
- **变量补全**：以$开头的变量名
- **文件补全**：包含路径字符或作为命令参数时

---

## 核心内容详解

### 主要导出类型

- **`ShellCompletionType`**
  - 类型：联合类型
  - 值：`'command' | 'variable' | 'file'`

### 主要导出函数

- **`getShellCompletions`**
  - 参数：
    - `input`（string）- 当前输入
    - `cursorOffset`（number）- 光标位置
    - `abortSignal`（AbortSignal）- 取消信号
  - 返回值：`Promise<SuggestionItem[]>`
  - 用途：主入口函数

### 补全命令生成

- **`getBashCompletionCommand`**
  - 参数：`prefix`, `completionType`
  - 用途：生成Bash compgen命令
  - 变量：`compgen -v`
  - 文件：`compgen -f` + 目录添加/后缀
  - 命令：`compgen -c`

- **`getZshCompletionCommand`**
  - 参数：`prefix`, `completionType`
  - 用途：生成Zsh原生命令
  - 变量：`print -rl -- ${(k)parameters[(I)prefix*]}`
  - 文件：zsh glob扩展
  - 命令：`print -rl -- ${(k)commands[(I)prefix*]}`

### 输入解析

- **`parseInputContext`**
  - 参数：`input`, `cursorOffset`
  - 用途：确定补全类型和前缀
  - 逻辑：
    - 以$开头 → 变量
    - 包含/或~ → 文件
    - 操作符后 → 命令
    - 否则 → 文件

---

## 设计要点

1. **Shell原生**：直接使用compgen而非维护独立数据库
2. **类型识别**：基于前缀和上下文智能判断补全类型
3. **超时保护**：1000ms超时防止Shell卡死
4. **数量限制**：最多15个补全结果

---

## 与其他文件的关系

- **依赖**：
  - `./shellQuote.ts` - 命令引用
  - `../Shell.ts` - 执行Shell命令
  - `../localInstaller.ts` - 获取Shell类型
- **被依赖**：
  - PromptInput组件 - 显示补全建议

---

## 注意事项

- 仅支持bash和zsh（与Shell.ts执行支持一致）
- 文件名包含换行符时使用while read处理
- Windows环境使用git bash的compgen
