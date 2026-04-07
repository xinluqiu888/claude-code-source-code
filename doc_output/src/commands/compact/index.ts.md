# compact/index.ts

## 文件描述
Compact 命令配置 - 压缩会话历史

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local |
| 名称 | compact |
| 描述 | Clear conversation history but keep a summary in context |
| 启用条件 | !DISABLE_COMPACT |
| 参数提示 | <optional custom summarization instructions> |

## 核心内容

### 命令配置
```typescript
const compact = {
  type: 'local',
  name: 'compact',
  description: 'Clear conversation history but keep a summary in context',
  isEnabled: () => !isEnvTruthy(process.env.DISABLE_COMPACT),
  supportsNonInteractive: true,
  argumentHint: '<optional custom summarization instructions>',
  load: () => import('./compact.js'),
} satisfies Command
```

## 设计点

1. **条件启用**：可通过 DISABLE_COMPACT 禁用
2. **非交互支持**：支持非交互模式
3. **可选参数**：支持自定义摘要指令
4. **懒加载**：动态导入实现

## 与其他文件的关系

- 导入 `isEnvTruthy` 从 `../../utils/envUtils.js`

## 注意事项

- 保留上下文摘要
- 支持非交互模式
- 可自定义摘要指令
