# config/index.ts

## 文件描述
Config 命令配置 - 打开配置面板

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | config |
| 别名 | settings |
| 描述 | Open config panel |
| 可用性 | 全局可用 |

## 核心内容

### 命令配置
```typescript
const config = {
  aliases: ['settings'],
  type: 'local-jsx',
  name: 'config',
  description: 'Open config panel',
  load: () => import('./config.js'),
} satisfies Command
```

## 设计点

1. **别名支持**：settings 别名便于用户理解
2. **全局可用**：所有环境都可以使用
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./config.js` 获取组件实现

## 注意事项

- 配置面板包含多项设置
- 支持持久化配置
