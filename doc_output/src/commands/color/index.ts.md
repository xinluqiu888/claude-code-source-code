# index.ts — color 命令入口配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/color/index.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 16 行 |
| 主要职责 | 定义 color 命令的元数据，支持即时执行模式 |

## 核心内容详解

### 命令配置

```typescript
const color = {
  type: 'local-jsx',
  name: 'color',
  description: 'Set the prompt bar color for this session',
  immediate: true,
  argumentHint: '<color|default>',
  load: () => import('./color.js'),
} satisfies Command
```

### 关键特性

| 属性 | 值 | 说明 |
|------|-----|------|
| `type` | `'local-jsx'` | 本地 JSX 组件命令 |
| `name` | `'color'` | 命令名 |
| `immediate` | `true` | 立即执行，不进入主消息循环 |
| `argumentHint` | `'<color|default>'` | 颜色名或 default |

## 设计要点

**即时模式 (`immediate: true`)**
- 设置颜色后立即生效
- 不需要等待主循环处理
- 适合配置类命令

## 与其他文件的关系

- **color.ts**: 实际的命令实现
