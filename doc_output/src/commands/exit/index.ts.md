# exit/index.ts

## 文件描述
Exit 命令配置 - 退出 REPL

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | exit |
| 别名 | quit |
| 描述 | Exit the REPL |
| 即时执行 | true |

## 核心内容

### 命令配置
```typescript
const exit = {
  type: 'local-jsx',
  name: 'exit',
  aliases: ['quit'],
  description: 'Exit the REPL',
  immediate: true,
  load: () => import('./exit.js'),
} satisfies Command
```

## 设计点

1. **多别名支持**：quit 别名便于用户习惯
2. **即时执行**：immediate: true 立即处理
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./exit.js` 获取组件实现

## 注意事项

- 退出当前会话
- 清理资源
- 保存状态
