# diff/index.ts

## 文件描述
Diff 命令配置 - 查看未提交的更改

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | diff |
| 描述 | View uncommitted changes and per-turn diffs |
| 可用性 | 全局可用 |

## 核心内容

### 命令配置
```typescript
export default {
  type: 'local-jsx',
  name: 'diff',
  description: 'View uncommitted changes and per-turn diffs',
  load: () => import('./diff.js'),
} satisfies Command
```

## 设计点

1. **简洁配置**：无别名，单一功能
2. **全局可用**：所有环境都可以使用
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./diff.js` 获取组件实现

## 注意事项

- 显示 Git 未提交更改
- 支持每轮对话的差异对比
