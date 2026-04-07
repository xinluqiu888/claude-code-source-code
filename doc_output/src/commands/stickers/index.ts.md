# stickers/index.ts

## 文件描述
Stickers 命令配置 - 订购 Claude Code 贴纸

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local |
| 名称 | stickers |
| 描述 | Order Claude Code stickers |
| 可用性 | 全局可用 |
| 功能 | 打开浏览器访问贴纸订购页面 |

## 函数概述

### 命令配置
配置贴纸订购命令：
- 简单的命令
- 打开外部链接

## 核心内容

### 命令配置
```typescript
{
  type: 'local',
  name: 'stickers',
  description: 'Order Claude Code stickers',
  availability: [GLOBAL_AVAILABILITY],
  load: () => import('./stickers.js'),
}
```

## 设计点

1. **简单配置**：local 类型，无复杂逻辑
2. **营销功能**：品牌推广用途
3. **外部链接**：打开浏览器访问订购页
4. **全局可用**：所有用户都可以访问

## 与其他文件的关系

- 导入 `./stickers.js` 获取命令实现
- 使用 `GLOBAL_AVAILABILITY` 定义全局可用性

## 注意事项

- 需要浏览器支持
- 链接可能因地区而异
- 可能需要跟踪点击统计
- 贴纸供应可能有限
