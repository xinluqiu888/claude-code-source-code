# gitignore.ts — Gitignore管理工具

> **一句话总结**：检查文件是否被gitignore忽略，并管理全局gitignore规则。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/git/gitignore.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~100行 |
| 主要职责 | gitignore检查和全局规则管理 |

---

## 功能概述

该模块提供：
1. 检查指定路径是否被gitignore（使用git check-ignore命令）
2. 向全局gitignore（~/.config/git/ignore）添加规则

全局gitignore用于添加跨项目的忽略规则，如编辑器临时文件等。

---

## 核心内容详解

### 导出函数

- **`isPathGitignored`**
  - 参数：`filePath`（string）, `cwd`（string）
  - 返回值：`Promise<boolean>`
  - 用途：检查路径是否被gitignore
  - 实现：执行`git check-ignore`
  - 返回值：exit code 0表示被忽略

- **`getGlobalGitignorePath`**
  - 返回值：`string`
  - 用途：获取全局gitignore文件路径
  - 路径：`~/.config/git/ignore`

- **`addFileGlobRuleToGitignore`**
  - 参数：`filename`（string）, `cwd`（string, 可选）
  - 用途：向全局gitignore添加规则
  - 行为：
    - 检查文件是否已被任何gitignore规则匹配
    - 已匹配则不添加
    - 使用`**/filename`模式

---

## 与其他文件的关系

- **依赖**：
  - `../execFileNoThrow.ts` - 执行git命令
  - `../git.ts` - 检查是否在git仓库
- **被依赖**：
  - 用于自动添加项目相关文件到gitignore

---

## 注意事项

- git check-ignore会检查所有gitignore源（项目.gitignore、.git/info/exclude、全局）
- 不在git仓库时返回false（失败开放）
- 目录模式以/结尾，检查时使用sample-file测试
