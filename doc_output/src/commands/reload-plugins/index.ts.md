# reload-plugins/index.ts

## 文件描述
Reload Plugins 命令配置 - 重新加载插件

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local |
| 名称 | reload-plugins |
| 描述 | Activate pending plugin changes in the current session |
| 支持非交互 | false |

## 核心内容

### 命令配置
```typescript
const reloadPlugins = {
  type: 'local',
  name: 'reload-plugins',
  description: 'Activate pending plugin changes in the current session',
  supportsNonInteractive: false,
  load: () => import('./reload-plugins.js'),
} satisfies Command
```

### 说明
- SDK 调用者使用 query.reloadPlugins()（控制请求）
- 返回结构化数据（commands, agents, plugins, mcpServers）用于 UI 更新

## 设计点

1. **Layer-3 刷新**：应用待处理的插件更改
2. **非交互限制**：不支持非交互模式
3. **懒加载**：动态导入实现

## 与其他文件的关系

- 导入 `./reload-plugins.js` 获取命令实现

## 注意事项

- 激活待处理的插件更改
- 影响当前会话
- SDK 使用控制请求替代
