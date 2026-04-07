# prefix.ts — 命令前缀静态分析器

> **一句话总结**：分析命令字符串提取命令前缀，支持包装命令递归解析和复合命令前缀折叠，用于权限系统的命令匹配。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/prefix.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~205行 |
| 主要职责 | 提取命令前缀，处理包装命令和复合命令 |

---

## 功能概述

该模块用于Claude Code的权限系统，帮助确定用户输入命令的分类前缀。例如：
- `git status` → 前缀 `git status`
- `docker run ubuntu` → 前缀 `docker run`
- `sudo systemctl restart` → 前缀 `systemctl restart`（穿透sudo包装）

主要特性：
1. **包装命令处理**：识别sudo、nice等包装命令，递归提取被包装命令
2. **复合命令支持**：处理`&&`、`||`、`;`分隔的多命令
3. **前缀折叠**：相同根命令的前缀会被折叠为最长公共前缀

---

## 核心内容详解

### 主要导出函数

- **`getCommandPrefixStatic`**
  - 参数：
    - `command`（string）- 要分析的命令
    - `recursionDepth`（number, 可选）- 递归深度，默认0
    - `wrapperCount`（number, 可选）- 包装命令计数，默认0
  - 返回值：`Promise<{ commandPrefix: string | null } | null>`
  - 用途：获取命令的前缀字符串
  - 限制：最大递归深度10，最大包装计数2

- **`getCompoundCommandPrefixesStatic`**
  - 参数：
    - `command`（string）- 复合命令
    - `excludeSubcommand`（函数, 可选）- 排除某些子命令的过滤器
  - 返回值：`Promise<string[]>`
  - 用途：处理复合命令（含&&/||/;），返回每个子命令的前缀
  - 特性：相同根命令的前缀会被折叠

### 包装命令处理

- **`WRAPPER_COMMANDS`**
  - 类型：Set<string>
  - 内容：`['nice']` - 特殊处理的包装命令
  - 说明：这些命令的选项处理复杂，无法通过spec表达

- **`handleWrapper`**（内部函数）
  - 用途：递归处理包装命令，提取被包装的命令前缀
  - 逻辑：查找spec中标记为`isCommand`的参数位置

### 辅助函数

- **`isKnownSubcommand`**
  - 用途：检查参数是否匹配已知的子命令
  - 作用：解决包装命令同时有子命令的歧义（如git）

- **`longestCommonPrefix`**
  - 用途：计算字符串数组的最长公共前缀（词边界对齐）
  - 示例：`["git fetch", "git worktree"]` → `"git"`

---

## 设计要点

1. **递归保护**：通过recursionDepth和wrapperCount防止无限递归
2. **Spec驱动**：使用命令spec（registry.ts）识别包装命令和子命令
3. **环境变量前缀**：保留环境变量赋值作为前缀的一部分
4. **智能折叠**：按根命令分组后计算最长公共前缀

---

## 与其他文件的关系

- **依赖**：
  - `./parser.ts` - 解析命令获取AST
  - `./registry.ts` - 获取命令spec
  - `../shell/specPrefix.ts` - 构建具体前缀
  - `./commands.ts` - 分割复合命令
- **被依赖**：
  - `../permissions/permissions.ts` - 权限匹配
  - `../readOnlyCommandValidation.ts` - 只读命令验证

---

## 注意事项

- 该模块用于权限系统的命令分类，不是通用命令解析器
- 包装命令识别依赖spec中的`isCommand`标记
- 复合命令分割使用commands.ts的splitCommand_DEPRECATED
- 最大递归深度和包装计数防止复杂输入导致的性能问题
