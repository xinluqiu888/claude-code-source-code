# ide/index.ts

## 文件描述
IDE 命令配置 - 管理 IDE 集成

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | ide |
| 描述 | Manage IDE integrations and show status |
| 参数提示 | [open] |

## 核心内容

### 命令配置
```typescript
const ide = {
  type: 'local-jsx',
  name: 'ide',
  description: 'Manage IDE integrations and show status',
  argumentHint: '[open]',
  load: () => import('./ide.js'),
} satisfies Command
```

## 设计点

1. **可选参数**：支持 open 参数
2. **集成管理**：管理 IDE 连接
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./ide.js` 获取组件实现

## 注意事项

- 支持多种 IDE
- 显示集成状态
