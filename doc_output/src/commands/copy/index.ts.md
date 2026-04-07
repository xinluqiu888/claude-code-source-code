# copy/index.ts

## 文件描述
Copy 命令配置 - 复制 Claude 的回复到剪贴板

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | copy |
| 描述 | Copy Claude's last response to clipboard |
| 可用性 | 全局可用 |

## 核心内容

### 命令配置
```typescript
const copy = {
  type: 'local-jsx',
  name: 'copy',
  description: "Copy Claude's last response to clipboard (or /copy N for the Nth-latest)",
  load: () => import('./copy.js'),
} satisfies Command
```

## 设计点

1. **便捷操作**：快速复制最近回复
2. **支持索引**：可指定第 N 条回复
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./copy.js` 获取组件实现

## 注意事项

- 需要剪贴板访问权限
- 支持指定历史消息
