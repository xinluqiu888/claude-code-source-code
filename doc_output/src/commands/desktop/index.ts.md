# index.ts — desktop 命令入口配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/desktop/index.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 26 行 |
| 主要职责 | 定义 desktop/app 命令的元数据和平台支持检查 |

## 核心内容详解

### 平台支持检查

```typescript
function isSupportedPlatform(): boolean {
  if (process.platform === 'darwin') {
    return true
  }
  if (process.platform === 'win32' && process.arch === 'x64') {
    return true
  }
  return false
}
```

支持的平台：
- macOS (`darwin`)
- Windows x64 (`win32` + `x64`)

### 命令配置

```typescript
const desktop = {
  type: 'local-jsx',
  name: 'desktop',
  aliases: ['app'],
  description: 'Continue the current session in Claude Desktop',
  availability: ['claude-ai'],
  isEnabled: isSupportedPlatform,
  get isHidden() {
    return !isSupportedPlatform()
  },
  load: () => import('./desktop.js'),
} satisfies Command
```

### 关键特性

| 属性 | 值 | 说明 |
|------|-----|------|
| `name` | `'desktop'` | 主命令名 |
| `aliases` | `['app']` | `/app` 别名 |
| `availability` | `['claude-ai']` | 仅 claude-ai 平台 |
| `isEnabled` | 函数 | 平台支持检查 |
| `isHidden` | getter | 不支持的平台隐藏 |

## 设计要点

1. **平台过滤**：不支持的平台自动隐藏命令
2. **别名友好**：`/app` 作为更短的别名
3. **平台限制**：仅支持 macOS 和 Windows x64
4. **订阅限制**：仅对 claude-ai 平台用户可用

## 与其他文件的关系

- **desktop.tsx**: 实际的切换界面实现
