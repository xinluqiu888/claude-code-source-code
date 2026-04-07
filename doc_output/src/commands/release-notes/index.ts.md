# release-notes/index.ts

## 文件描述
Release Notes 命令配置 - 查看发布说明

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local |
| 名称 | release-notes |
| 描述 | View release notes |
| 支持非交互 | true |

## 核心内容

### 命令配置
```typescript
const releaseNotes: Command = {
  description: 'View release notes',
  name: 'release-notes',
  type: 'local',
  supportsNonInteractive: true,
  load: () => import('./release-notes.js'),
}
```

## 设计点

1. **简洁配置**：基础功能
2. **非交互支持**：支持非交互模式
3. **懒加载**：动态导入实现

## 与其他文件的关系

- 导入 `./release-notes.js` 获取命令实现

## 注意事项

- 显示版本发布说明
- 包含更新内容
