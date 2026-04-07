# permissions/index.ts

## 文件描述
Permissions 命令配置 - 管理工具权限规则

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | permissions |
| 别名 | allowed-tools |
| 描述 | Manage allow & deny tool permission rules |
| 可用性 | 全局可用 |

## 核心内容

### 命令配置
```typescript
const permissions = {
  type: 'local-jsx',
  name: 'permissions',
  aliases: ['allowed-tools'],
  description: 'Manage allow & deny tool permission rules',
  load: () => import('./permissions.js'),
} satisfies Command
```

## 设计点

1. **多别名支持**：allowed-tools 别名便于理解
2. **权限管理**：集中管理工具权限
3. **全局可用**：所有环境都可以使用
4. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./permissions.js` 获取组件实现

## 注意事项

- 管理允许/拒绝规则
- 影响工具使用权限
