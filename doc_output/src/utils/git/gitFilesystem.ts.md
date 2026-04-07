# gitFilesystem.ts — Git文件系统状态读取

> **一句话总结**：通过直接读取.git目录文件获取仓库状态，避免启动git子进程。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/git/gitFilesystem.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~700行 |
| 主要职责 | 无git命令的仓库状态读取 |

---

## 功能概述

该模块实现高效的Git状态读取，直接操作.git目录下的文件：
- 解析HEAD文件获取当前分支或分离HEAD的SHA
- 读取refs/目录解析引用
- 读取packed-refs处理打包引用
- 支持worktree和submodule

通过`GitFileWatcher`类提供缓存和文件监视功能，避免重复读取。

---

## 核心内容详解

### 导出函数

- **`resolveGitDir`** - 解析.git目录（支持worktree的gitdir文件）
- **`readGitHead`** - 读取HEAD文件
- **`resolveRef`** - 解析ref到SHA
- **`getCommonDir`** - 获取worktree的common目录
- **`isShallowClone`** - 检查是否为浅克隆
- **`getWorktreeCountFromFs`** - 统计worktree数量

### 缓存函数

- **`getCachedBranch`** - 获取当前分支（带缓存）
- **`getCachedHead`** - 获取HEAD SHA（带缓存）
- **`getCachedRemoteUrl`** - 获取remote url（带缓存）
- **`getCachedDefaultBranch`** - 获取默认分支（带缓存）

### GitFileWatcher类

- 监视文件：HEAD、config、当前分支的ref文件
- 文件变化时自动使缓存失效
- 分支切换时更新分支ref监视

### 安全函数

- **`isSafeRefName`** - 验证ref名安全（防止路径遍历和shell注入）
- **`isValidGitSha`** - 验证SHA格式（40或64位十六进制）

---

## 与其他文件的关系

- **依赖**：
  - `./gitConfigParser.ts` - 解析config文件
  - `../git.ts` - findGitRoot
- **被依赖**：
  - 多处用于快速获取Git状态

---

## 注意事项

- ref名和SHA在使用前必须通过安全验证
- worktree配置在commondir文件中
- packed-refs格式需按packed-backend.c处理
