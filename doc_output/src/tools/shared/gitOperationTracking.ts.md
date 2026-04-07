# gitOperationTracking.ts — Git 操作跟踪

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/tools/shared/gitOperationTracking.ts`
- **类型**: TypeScript 共享模块
- **功能**: 跟踪 Git 操作以进行使用指标分析

## 核心内容详解

### 导出函数

```typescript
export function detectGitOperation(
  command: string,
  output: string
): {
  commit?: { sha: string; kind: CommitKind }
  push?: { branch: string }
  branch?: { ref: string; action: BranchAction }
  pr?: { number: number; url?: string; action: PrAction }
}

export function trackGitOperations(
  command: string,
  exitCode: number,
  stdout?: string
): void

// 解析函数（用于测试）
export function parseGitCommitId(stdout: string): string | undefined
export function parsePrUrl(url: string): { prNumber: number; prUrl: string; prRepository: string } | null
```

### 类型定义

```typescript
export type CommitKind = 'committed' | 'amended' | 'cherry-picked'
export type BranchAction = 'merged' | 'rebased'
export type PrAction = 
  | 'created' | 'edited' | 'merged' | 'commented' | 'closed' | 'ready'
```

### Git 命令正则

```typescript
// 构建 git 命令正则（支持全局选项）
function gitCmdRe(subcmd: string, suffix = ''): RegExp

const GIT_COMMIT_RE = gitCmdRe('commit')
const GIT_PUSH_RE = gitCmdRe('push')
const GIT_CHERRY_PICK_RE = gitCmdRe('cherry-pick')
const GIT_MERGE_RE = gitCmdRe('merge', '(?!-)')
const GIT_REBASE_RE = gitCmdRe('rebase')
```

### GitHub PR 操作

```typescript
const GH_PR_ACTIONS = [
  { re: /\bgh\s+pr\s+create\b/, action: 'created', op: 'pr_create' },
  { re: /\bgh\s+pr\s+edit\b/, action: 'edited', op: 'pr_edit' },
  { re: /\bgh\s+pr\s+merge\b/, action: 'merged', op: 'pr_merge' },
  { re: /\bgh\s+pr\s+comment\b/, action: 'commented', op: 'pr_comment' },
  { re: /\bgh\s+pr\s+close\b/, action: 'closed', op: 'pr_close' },
  { re: /\bgh\s+pr\s+ready\b/, action: 'ready', op: 'pr_ready' },
]
```

### 解析函数

#### parseGitCommitId
从 git commit 输出解析提交 ID：
```
[branch abc1234] message
[branch (root-commit) abc1234] message
```

#### parseGitPushBranch
从 git push 输出解析分支名：
```
abc..def  branch -> branch
* [new branch] branch -> branch
+ abc...def  branch -> branch (forced update)
```

#### parsePrUrl
解析 GitHub PR URL：
```
https://github.com/owner/repo/pull/123
```

#### parsePrNumberFromText
从文本提取 PR 号：
```
pull request owner/repo#123
```

### 检测逻辑

#### detectGitOperation
扫描命令和输出以检测 Git 操作：

| 操作 | 检测条件 |
|------|----------|
| commit | `git commit` 命令 + `[branch sha]` 输出 |
| cherry-pick | `git cherry-pick` 命令 |
| amend | `git commit` 命令 + `--amend` 参数 |
| push | `git push` 命令 + 分支更新输出 |
| merge | `git merge` 命令 + 成功消息 |
| rebase | `git rebase` 命令 + 成功消息 |
| PR | `gh pr` 命令 + PR URL 在输出中 |

### 跟踪逻辑

#### trackGitOperations
成功执行时（exitCode === 0）：

1. **提交**: 发送 `tengu_git_operation` 事件（operation: commit/commit_amend）
2. **推送**: 发送 `tengu_git_operation` 事件（operation: push）
3. **PR 操作**: 发送 `tengu_git_operation` 事件（对应操作类型）
4. **PR 创建**: 增加 PR 计数器，自动链接会话到 PR
5. **glab**: 支持 GitLab mr create
6. **curl**: 检测通过 curl 创建 PR

## 设计要点

### 跨 Shell 兼容
- 正则表达式在原始命令文本上操作
- 对 Bash 和 PowerShell 同样有效（都使用相同的外部二进制语法）

### Git 全局选项
- 支持 `-c key=val`, `-C path`, `--git-dir=path` 等全局选项
- 使用 `gitCmdRe()` 构建正则表达式

### 动态导入
- 使用 `import()` 动态导入避免循环依赖
- 用于会话存储和状态获取

## 与其他文件的关系

### 依赖
- `../../bootstrap/state.js` — 提交和 PR 计数器
- `../../services/analytics/index.js` — 分析事件

### 被依赖
- BashTool 等执行 Git 命令的工具
