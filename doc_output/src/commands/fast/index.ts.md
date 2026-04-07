# fast/index.ts

## 文件描述
Fast 命令配置 - 切换快速模式

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | fast |
| 描述 | 动态描述（显示当前模型） |
| 可用性 | claude-ai, console |
| 参数提示 | [on|off] |

## 核心内容

### 命令配置
```typescript
const fast = {
  type: 'local-jsx',
  name: 'fast',
  get description() {
    return `Toggle fast mode (${FAST_MODE_MODEL_DISPLAY} only)`;
  },
  availability: ['claude-ai', 'console'],
  isEnabled: () => isFastModeEnabled(),
  get isHidden() { return !isFastModeEnabled(); },
  argumentHint: '[on|off]',
  get immediate() {
    return shouldInferenceConfigCommandBeImmediate();
  },
  load: () => import('./fast.js'),
} satisfies Command
```

## 设计点

1. **动态描述**：显示当前快速模式模型
2. **条件启用**：根据功能标志启用
3. **即时执行**：动态计算执行时机
4. **环境限制**：仅特定环境可用

## 与其他文件的关系

- 导入快速模式工具
- 使用即时命令配置

## 注意事项

- 快速模式使用轻量级模型
- 仅 claude-ai 和 console 环境可用
