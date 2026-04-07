# theme/index.ts

## 文件描述
Theme 命令配置 - 更改主题

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | theme |
| 描述 | Change the theme |
| 可用性 | 全局可用 |

## 核心内容

### 命令配置
```typescript
const theme = {
  type: 'local-jsx',
  name: 'theme',
  description: 'Change the theme',
  load: () => import('./theme.js'),
} satisfies Command
```

## 设计点

1. **简洁配置**：无别名，单一功能
2. **全局可用**：所有环境都可以使用
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./theme.js` 获取组件实现

## 注意事项

- 切换界面主题
- 支持多种主题
