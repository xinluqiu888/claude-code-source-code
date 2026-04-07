# add-dir.tsx — 添加工作目录的交互式命令实现

> **一句话总结**：提供 `/add-dir` 斜杠命令的 React 渲染逻辑，支持有路径参数和无路径参数两种添加工作目录的交互方式。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/commands/add-dir/add-dir.tsx` |
| 文件类型 | TSX（React 组件 + 异步命令入口） |
| 代码行数 | 约 200 行（编译后） |
| 主要职责 | 实现 `/add-dir` 命令的 UI 交互与目录权限更新逻辑 |

---

## 功能概述

`add-dir.tsx` 是 `/add-dir` 命令的核心实现文件，负责处理用户在 Claude Code REPL 中执行 `/add-dir <路径>` 时的完整交互流程。

当用户提供路径参数时，文件先调用 `validateDirectoryForWorkspace` 验证该路径是否合法，然后通过 `AddWorkspaceDirectory` 组件（一个权限确认 UI）让用户选择是否仅对当前会话生效，或永久保存到本地配置。

当用户未提供路径参数时，同样渲染 `AddWorkspaceDirectory` 组件，由组件内部的交互式选择器引导用户输入路径。

目录被添加后，代码会同步更新 `toolPermissionContext`（内存状态），并通过 `SandboxManager.refreshConfig()` 刷新沙箱配置，确保 Bash 工具可以立即访问新目录，避免竞态条件。

---

## 核心内容详解

### 导入与依赖

- **`validation.ts`**：提供 `validateDirectoryForWorkspace` 和 `addDirHelpMessage`，用于路径合法性检查与错误信息生成
- **`AddWorkspaceDirectory`**（组件）：可重用的工作目录确认对话框
- **`applyPermissionUpdate` / `persistPermissionUpdate`**：权限更新的内存应用与持久化
- **`SandboxManager`**：刷新沙箱配置，使新目录对 Bash 工具生效
- **`getAdditionalDirectoriesForClaudeMd` / `setAdditionalDirectoriesForClaudeMd`**：管理 bootstrap 层的附加目录状态

### 主要类/函数/接口

#### `AddDirError` 组件（内部）
错误回显组件，展示 `/add-dir <args>` 命令提示及错误信息，并在挂载后立即通过 `setTimeout(onDone, 0)` 触发完成回调，实现即时退出。

#### `handleAddDirectory(path, remember)` 闭包（内部）
核心添加逻辑：
1. 根据 `remember` 参数决定目的地（`'localSettings'` 或 `'session'`）
2. 构造 `permissionUpdate` 对象，类型为 `'addDirectories'`
3. 调用 `applyPermissionUpdate` 更新会话上下文
4. 更新 bootstrap 状态中的附加目录列表
5. 调用 `SandboxManager.refreshConfig()` 刷新沙箱
6. 若 `remember` 为 `true`，调用 `persistPermissionUpdate` 持久化到本地配置

#### `call(onDone, context, args)` — 导出的命令入口
异步函数，返回 React 节点：
- 若 args 非空：先验证路径，验证失败则渲染 `AddDirError`；验证成功则渲染 `AddWorkspaceDirectory`
- 若 args 为空：直接渲染 `AddWorkspaceDirectory`（无需预校验）

### 数据流与逻辑流程

```
用户输入 /add-dir [path]
    ↓
call() 函数被调用
    ↓
有 args → validateDirectoryForWorkspace()
    ├── 失败 → AddDirError 组件（展示错误，立即退出）
    └── 成功 → AddWorkspaceDirectory 组件（确认对话框）
无 args → AddWorkspaceDirectory 组件（引导输入）
    ↓
用户确认后 → handleAddDirectory()
    ↓
更新内存权限 → 更新 bootstrap 状态 → 刷新沙箱 → [可选] 持久化
    ↓
onDone() 回调结束命令
```

### 对外接口（导出内容）

- **`call`**：`async function call(onDone, context, args?): Promise<React.ReactNode>`
  命令执行入口，由命令系统通过 `load()` 动态导入后调用。

---

## 设计要点

- **React Compiler 优化**：文件使用 `_c` 缓存机制（React Compiler 自动生成），对 props 进行细粒度缓存，避免不必要的重渲染
- **即时沙箱刷新**：`SandboxManager.refreshConfig()` 在添加目录后立即调用，避免用户立刻执行 Bash 命令时访问权限滞后
- **两种持久化级别**：`session`（仅本次会话）和 `localSettings`（保存到 `.claude/settings.local.json`），通过 `remember` 参数区分

---

## 与其他文件的关系

- **依赖**：
  - `./validation.ts`：路径验证与帮助信息生成
  - `../../components/permissions/rules/AddWorkspaceDirectory.js`：UI 组件
  - `../../utils/permissions/PermissionUpdate.js`：权限更新工具函数
  - `../../utils/sandbox/sandbox-adapter.js`：沙箱配置管理
  - `../../bootstrap/state.js`：全局状态管理
- **被依赖**：
  - `./index.ts`：通过 `load: () => import('./add-dir.js')` 懒加载此文件

---

## 注意事项

- 该文件是编译后的 TSX（使用了 React Compiler 的 `_c` 运行时），阅读时需注意 `$` 数组为编译器生成的缓存槽位，不是业务逻辑
- `handleAddDirectory` 中对 `getAdditionalDirectoriesForClaudeMd` 的检查（`!currentDirs.includes(path)`）防止重复添加同一路径
- `persistPermissionUpdate` 调用失败时有 `try-catch` 容错处理，错误信息会通过 `onDone` 回调展示给用户
