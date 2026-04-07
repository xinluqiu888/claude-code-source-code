# privacy-settings/index.ts

## 文件描述
Privacy Settings 命令配置 - 查看和更新隐私设置

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | privacy-settings |
| 描述 | View and update your privacy settings |
| 启用条件 | isConsumerSubscriber() |

## 核心内容

### 命令配置
```typescript
const privacySettings = {
  type: 'local-jsx',
  name: 'privacy-settings',
  description: 'View and update your privacy settings',
  isEnabled: () => isConsumerSubscriber(),
  load: () => import('./privacy-settings.js'),
} satisfies Command
```

## 设计点

1. **订阅者专用**：仅消费者订阅者可用
2. **隐私控制**：管理隐私设置
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `isConsumerSubscriber` 从 `../../utils/auth.js`

## 注意事项

- 仅订阅者可访问
- 涉及数据隐私设置
