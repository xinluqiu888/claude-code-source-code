# rewind/index.ts

## 文件描述
Rewind 命令配置 - 恢复到之前状态

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local |
| 名称 | rewind |
| 别名 | checkpoint |
| 描述 | Restore the code and/or conversation to a previous point |
| 支持非交互 | false |

## 核心内容

### 命令配置
```typescript
const rewind = {
  description: 'Restore the code and/or conversation to a previous point',
  name: 'rewind',
  aliases: ['checkpoint'],
  argumentHint: '',
  type: 'local',
  supportsNonInteractive: false,
  load: () => import('./rewind.js'),
} satisfies Command
```

## 设计点

1. **多别名支持**：checkpoint 别名便于理解
2. **恢复功能**：代码和/或对话恢复
3. **交互必需**：不支持非交互模式
4. **懒加载**：动态导入实现

## 与其他文件的关系

- 导入 `./rewind.js` 获取命令实现

## 注意事项

- 恢复到之前的检查点
- 支持代码和对话恢复
- 需要交互式环境
