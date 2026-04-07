# effort/index.ts

## 文件描述
Effort 命令配置 - 设置模型使用努力级别

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | effort |
| 描述 | Set effort level for model usage |
| 参数提示 | [low|medium|high|max|auto] |
| 即时执行 | 动态计算 |

## 核心内容

### 命令配置
```typescript
export default {
  type: 'local-jsx',
  name: 'effort',
  description: 'Set effort level for model usage',
  argumentHint: '[low|medium|high|max|auto]',
  get immediate() {
    return shouldInferenceConfigCommandBeImmediate();
  },
  load: () => import('./effort.js'),
} satisfies Command
```

### 即时执行
- 使用 `shouldInferenceConfigCommandBeImmediate()` 动态计算
- 根据配置决定是否立即执行

## 设计点

1. **动态即时性**：根据配置决定执行时机
2. **参数提示**：提供可选值提示
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `shouldInferenceConfigCommandBeImmediate` 从 `../../utils/immediateCommand.js`

## 注意事项

- 影响模型响应深度
- 可选值：low, medium, high, max, auto
