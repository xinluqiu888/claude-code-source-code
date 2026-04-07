# bundledSkills.ts — 捆绑技能管理器

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/skills/bundledSkills.ts`
- **类型**: TypeScript 模块
- **作用**: 管理随CLI一起分发的捆绑技能（built-in skills）

## 功能概述

本模块提供了一套API来注册和管理捆绑技能——这些技能编译在CLI二进制文件中，对所有用户可用。与从文件系统加载的技能不同，捆绑技能是程序化的，支持引用文件的延迟提取。

## 核心内容详解

### 类型定义

```typescript
export type BundledSkillDefinition = {
  name: string
  description: string
  aliases?: string[]           // 别名
  whenToUse?: string          // 使用时机说明
  argumentHint?: string       // 参数提示
  allowedTools?: string[]     // 允许的工具
  model?: string              // 指定模型
  disableModelInvocation?: boolean
  userInvocable?: boolean     // 用户是否可调用
  isEnabled?: () => boolean   // 动态启用检查
  hooks?: HooksSettings       // 钩子配置
  context?: 'inline' | 'fork' // 执行上下文
  agent?: string              // Agent类型
  files?: Record<string, string> // 引用文件（延迟提取）
  getPromptForCommand: (args, context) => Promise<ContentBlockParam[]>
}
```

### 主要函数

#### `registerBundledSkill(definition: BundledSkillDefinition): void`
注册一个新的捆绑技能。

**文件提取机制**：
- 如果定义包含`files`字段，会在首次调用时提取到磁盘
- 提取目录：`~/.claude/.bundled-skills/<skillName>/`
- 使用O_NOFOLLOW|O_EXCL安全写入，防止符号链接攻击
- 目录权限0o700，文件权限0o600

#### `getBundledSkills(): Command[]`
获取所有已注册的捆绑技能（返回副本防止外部修改）。

#### `getBundledSkillExtractDir(skillName: string): string`
获取技能文件提取目录路径。

### 文件提取安全

```typescript
// 安全写入标志
const SAFE_WRITE_FLAGS =
  process.platform === 'win32'
    ? 'wx'  // Windows使用字符串标志
    : O_WRONLY | O_CREAT | O_EXCL | O_NOFOLLOW  // Unix使用数值标志
```

**安全措施**：
1. **O_NOFOLLOW**: 不跟随符号链接
2. **O_EXCL**: 文件存在时失败（防止竞态条件）
3. **0o700/0o600权限**: 即使umask=0也保持owner-only访问
4. **路径验证**: 使用`resolveSkillFilePath`防止目录遍历攻击

## 设计要点

1. **延迟提取**: 文件只在首次技能调用时提取，避免不必要的IO
2. **Promise记忆化**: 并发调用共享同一个提取Promise，避免竞态写入
3. **基础目录前缀**: 提取后，prompt前缀添加"Base directory for this skill: <dir>"
4. **与磁盘技能一致**: 捆绑技能可以通过Read/Grep访问提取的文件

## 与其他文件的关系

- **被 builtinPlugins.ts 使用**: 将内置插件的技能转换为Command对象
- **使用 filesystem.ts**: `getBundledSkillsRoot()`获取提取根目录
- **使用 debug.ts**: 记录提取失败信息

## 注意事项

1. **提取失败处理**: 如果提取失败，技能仍然可用，只是没有基础目录前缀
2. **路径规范化**: 技能文件路径使用正斜杠，通过`normalize`和`resolveSkillFilePath`验证
3. **清理**: `clearBundledSkills()`用于测试清理注册表
4. **权限防御**: 每进程随机数（nonce）作为主要防御，显式模式作为备份
