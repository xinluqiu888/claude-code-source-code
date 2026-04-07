# gitSafety.ts — Git安全验证

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/PowerShellTool/gitSafety.ts`
- **作用**: 防止通过Git命令进行沙箱逃逸

## 核心内容详解

### 安全威胁

Git可通过两种向量化武器进行沙箱逃逸：

1. **裸仓库攻击**: 如果cwd包含HEAD + objects/ + refs/但无有效的.git/HEAD，Git将cwd视为裸仓库并从cwd运行hooks
2. **Git内部写入 + git**: 复合命令创建HEAD/objects/refs/hooks/然后运行git —— git子命令执行刚创建的恶意hooks

### 路径处理

```typescript
// 解决cwd重新进入
function resolveCwdReentry(normalized: string): string
```

处理`../<cwd-basename>/`重新进入cwd的情况。

### 路径规范化

```typescript
function normalizeGitPathArg(arg: string): string {
  // 1. 移除参数前缀
  // 2. 移除引号
  // 3. 移除backtick转义
  // 4. 移除FileSystem::前缀
  // 5. 处理盘符相对路径
  // 6. 统一正斜杠
  // 7. 处理Win32 CreateFileW行为
  // 8. posix.normalize
}
```

### Git内部路径检测

```typescript
const GIT_INTERNAL_PREFIXES = ['head', 'objects', 'refs', 'hooks'] as const

export function isGitInternalPathPS(arg: string): boolean
export function isDotGitPathPS(arg: string): boolean
```

## 设计要点

1. **多层防护**: 覆盖裸仓库和标准仓库路径
2. **路径解析**: 处理相对路径和绝对路径
3. **大小写不敏感**: Windows路径比较
4. **NTFS 8.3**: 处理短名称（GIT~1）

## 与其他文件的关系

- **pathValidation.ts**: 调用进行Git路径验证
