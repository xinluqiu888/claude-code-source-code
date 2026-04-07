# index.ts — clear 命令入口配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/clear/index.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 19 行 |
| 主要职责 | 定义 clear 命令的元数据、别名和懒加载配置 |

## 核心内容详解

### 命令配置

```typescript
const clear = {
  type: 'local',
  name: 'clear',
  description: 'Clear conversation history and free up context',
  aliases: ['reset', 'new'],
  supportsNonInteractive: false,
  load: () => import('./clear.js'),
} satisfies Command
```

### 关键特性

| 属性 | 值 | 说明 |
|------|-----|------|
| `type` | `'local'` | 本地命令（非 JSX） |
| `name` | `'clear'` | 主命令名 |
| `aliases` | `['reset', 'new']` | 两个常用别名 |
| `supportsNonInteractive` | `false` | 非交互式会话不支持 |
| `load` | 函数 | 懒加载到 clear.ts |

### 注释说明

文件注释说明：
- 实现从 `clear.ts` 懒加载
- `clearSessionCaches` 从 `./clear/caches.js` 导入
- `clearConversation` 从 `./clear/conversation.js` 导入

## 设计要点

1. **多个别名**：提供 `reset` 和 `new` 作为常用替代名称
2. **非交互式限制**：在非交互式会话中应该创建新会话而非清除
3. **模块分离**：将缓存清理和对话清理分离到不同文件

## 与其他文件的关系

- **clear.ts**: 本地命令实现
- **caches.ts**: 缓存清理工具
- **conversation.ts**: 对话清理核心逻辑
