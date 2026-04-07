# help/index.ts

## 文件描述
Help 命令配置 - 显示帮助信息

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | help |
| 描述 | Show help and available commands |
| 可用性 | 全局可用 |

## 核心内容

### 命令配置
```typescript
const help = {
  type: 'local-jsx',
  name: 'help',
  description: 'Show help and available commands',
  load: () => import('./help.js'),
} satisfies Command
```

## 设计点

1. **简洁配置**：无别名，基础功能
2. **全局可用**：所有环境都可以使用
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./help.js` 获取组件实现

## 注意事项

- 显示可用命令列表
- 提供命令使用说明
