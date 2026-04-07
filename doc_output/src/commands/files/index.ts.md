# files/index.ts

## 文件描述
Files 命令配置 - 列出上下文中的文件

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local |
| 名称 | files |
| 描述 | List all files currently in context |
| 启用条件 | USER_TYPE === 'ant' |
| 支持非交互 | true |

## 核心内容

### 命令配置
```typescript
const files = {
  type: 'local',
  name: 'files',
  description: 'List all files currently in context',
  isEnabled: () => process.env.USER_TYPE === 'ant',
  supportsNonInteractive: true,
  load: () => import('./files.js'),
} satisfies Command
```

## 设计点

1. **Ant 专用**：仅内部用户可用
2. **非交互支持**：支持非交互模式
3. **懒加载**：动态导入实现

## 与其他文件的关系

- 导入 `./files.js` 获取命令实现

## 注意事项

- 仅 Ant 内部用户使用
- 显示当前上下文中的所有文件
