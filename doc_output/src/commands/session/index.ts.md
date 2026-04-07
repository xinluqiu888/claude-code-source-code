# session/index.ts

## 文件描述
Session 命令配置 - 远程会话管理

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | session |
| 描述 | Show a URL and QR code for the remote session |
| 可用性 | claude-ai |
| 功能 | 显示远程会话的URL和二维码 |

## 函数概述

### 命令配置
配置远程会话命令：
- 仅用于显示远程会话信息
- 生成URL和二维码供移动端连接

## 核心内容

### 命令配置
```typescript
{
  type: 'local-jsx',
  name: 'session',
  description: 'Show a URL and QR code for the remote session',
  availability: ['claude-ai'],
  load: () => import('./session.js'),
}
```

## 设计点

1. **简洁配置**：无复杂参数，单一功能
2. **环境限制**：仅在 claude-ai 环境可用
3. **移动连接**：支持移动端扫码连接
4. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `./session.js` 获取组件实现
- 依赖 `Command` 类型定义

## 注意事项

- 仅在 claude-ai 环境可用
- 需要远程会话服务支持
- 二维码生成需要额外依赖
- URL 需要包含会话认证信息
