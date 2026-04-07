# tag/index.ts

## 文件描述
Tag 命令配置 - 切换可搜索标签

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | tag |
| 描述 | Toggle a searchable tag on the current session |
| 启用条件 | USER_TYPE === 'ant' |
| 参数提示 | <tag-name> |

## 核心内容

### 命令配置
```typescript
const tag = {
  type: 'local-jsx',
  name: 'tag',
  description: 'Toggle a searchable tag on the current session',
  isEnabled: () => process.env.USER_TYPE === 'ant',
  argumentHint: '<tag-name>',
  load: () => import('./tag.js'),
} satisfies Command
```

## 设计点

1. **Ant 专用**：仅内部用户可用
2. **标签管理**：添加/移除会话标签
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./tag.js` 获取组件实现

## 注意事项

- 仅 Ant 内部用户使用
- 用于会话标记和搜索
