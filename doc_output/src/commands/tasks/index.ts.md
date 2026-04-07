# tasks/index.ts

## 文件描述
Tasks 命令配置 - 列出和管理后台任务

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | tasks |
| 别名 | bashes |
| 描述 | List and manage background tasks |
| 可用性 | 全局可用 |

## 核心内容

### 命令配置
```typescript
const tasks = {
  type: 'local-jsx',
  name: 'tasks',
  aliases: ['bashes'],
  description: 'List and manage background tasks',
  load: () => import('./tasks.js'),
} satisfies Command
```

## 设计点

1. **多别名支持**：bashes 别名便于理解
2. **任务管理**：列出和管理后台任务
3. **全局可用**：所有环境都可以使用
4. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./tasks.js` 获取组件实现

## 注意事项

- 管理后台任务
- 显示任务状态
- 支持任务操作
