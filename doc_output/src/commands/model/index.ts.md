# model/index.ts

## 文件描述
Model 命令配置 - 设置 AI 模型

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | model |
| 描述 | 动态描述（显示当前模型） |
| 参数提示 | [model] |
| 即时执行 | 动态计算 |

## 核心内容

### 命令配置
```typescript
export default {
  type: 'local-jsx',
  name: 'model',
  get description() {
    return `Set the AI model for Claude Code (currently ${renderModelName(getMainLoopModel())})`;
  },
  argumentHint: '[model]',
  get immediate() {
    return shouldInferenceConfigCommandBeImmediate();
  },
  load: () => import('./model.js'),
} satisfies Command
```

### 动态描述
- 显示当前使用的模型名称

## 设计点

1. **动态描述**：实时显示当前模型
2. **即时执行**：动态计算执行时机
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入模型工具函数
- 使用即时命令配置

## 注意事项

- 影响后续对话的模型选择
- 需要有效的模型名称
