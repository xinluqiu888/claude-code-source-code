# desktop/index.ts

## 文件描述
Desktop 命令配置 - 在 Claude Desktop 中继续会话

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | desktop |
| 别名 | app |
| 描述 | Continue the current session in Claude Desktop |
| 可用性 | claude-ai |
| 启用条件 | macOS 或 Windows x64 |

## 核心内容

### 命令配置
```typescript
const desktop = {
  type: 'local-jsx',
  name: 'desktop',
  aliases: ['app'],
  description: 'Continue the current session in Claude Desktop',
  availability: ['claude-ai'],
  isEnabled: isSupportedPlatform,
  get isHidden() { return !isSupportedPlatform(); },
  load: () => import('./desktop.js'),
} satisfies Command
```

### 平台支持
```typescript
function isSupportedPlatform(): boolean {
  if (process.platform === 'darwin') return true;
  if (process.platform === 'win32' && process.arch === 'x64') return true;
  return false;
}
```

## 设计点

1. **平台限制**：仅支持 macOS 和 Windows x64
2. **条件隐藏**：不支持的平台隐藏命令
3. **别名设计**：app 别名便于记忆
4. **环境限制**：仅在 claude-ai 可用

## 与其他文件的关系

- 导入 `./desktop.js` 获取组件实现

## 注意事项

- 需要 Claude Desktop 已安装
- 仅特定平台支持
- 需要 claude-ai 认证
