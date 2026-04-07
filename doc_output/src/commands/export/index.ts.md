# export/index.ts

## 文件描述
Export 命令配置 - 导出会话到文件或剪贴板

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | export |
| 描述 | Export the current conversation to a file or clipboard |
| 参数提示 | [filename] |

## 核心内容

### 命令配置
```typescript
const exportCommand = {
  type: 'local-jsx',
  name: 'export',
  description: 'Export the current conversation to a file or clipboard',
  argumentHint: '[filename]',
  load: () => import('./export.js'),
} satisfies Command
```

## 设计点

1. **可选参数**：文件名可选，支持剪贴板
2. **灵活导出**：支持文件和剪贴板
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./export.js` 获取组件实现

## 注意事项

- 支持自定义文件名
- 默认导出到剪贴板
