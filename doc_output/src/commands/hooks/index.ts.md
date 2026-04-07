# hooks/index.ts

## 文件描述
Hooks 命令配置 - 查看钩子配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | hooks |
| 描述 | View hook configurations for tool events |
| 即时执行 | true |

## 核心内容

### 命令配置
```typescript
const hooks = {
  type: 'local-jsx',
  name: 'hooks',
  description: 'View hook configurations for tool events',
  immediate: true,
  load: () => import('./hooks.js'),
} satisfies Command
```

## 设计点

1. **即时执行**：immediate: true 立即显示
2. **工具事件**：查看钩子配置
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./hooks.js` 获取组件实现

## 注意事项

- 显示工具事件钩子
- 支持会话生命周期钩子
