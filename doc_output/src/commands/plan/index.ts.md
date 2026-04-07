# plan/index.ts

## 文件描述
Plan 命令配置 - 启用计划模式

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | plan |
| 描述 | Enable plan mode or view the current session plan |
| 参数提示 | [open|<description>] |

## 核心内容

### 命令配置
```typescript
const plan = {
  type: 'local-jsx',
  name: 'plan',
  description: 'Enable plan mode or view the current session plan',
  argumentHint: '[open|<description>]',
  load: () => import('./plan.js'),
} satisfies Command
```

## 设计点

1. **参数灵活**：支持 open 或描述
2. **计划模式**：结构化对话
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./plan.js` 获取组件实现

## 注意事项

- 启用结构化计划模式
- 支持查看当前计划
