# index.ts — config 命令入口配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/config/index.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 11 行 |
| 主要职责 | 定义 config/settings 命令的元数据 |

## 核心内容详解

### 命令配置

```typescript
const config = {
  aliases: ['settings'],
  type: 'local-jsx',
  name: 'config',
  description: 'Open config panel',
  load: () => import('./config.js'),
} satisfies Command
```

### 关键特性

| 属性 | 值 | 说明 |
|------|-----|------|
| `name` | `'config'` | 主命令名 |
| `aliases` | `['settings']` | `/settings` 别名 |
| `type` | `'local-jsx'` | 本地 JSX 组件 |
| `description` | 打开配置面板 | 功能描述 |

## 设计要点

1. **别名友好**：`settings` 是更直观的别名
2. **简单配置**：无特殊启用条件
3. **配置集中**：统一的配置管理入口

## 与其他文件的关系

- **config.tsx**: 实际的配置面板实现
