# stickers/stickers.ts

## 文件描述
Stickers 命令实现 - 打开贴纸订购页面

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | Local命令 |
| 导出 | call函数 |
| 功能 | 打开浏览器到贴纸订购页面 |
| URL | https://www.stickermule.com/claudecode |

## 函数概述

### call
执行打开浏览器操作：
1. 调用 openBrowser 函数
2. 访问 stickermule 贴纸页面

### 函数签名
```typescript
export const call: LocalCommandCall = async () => {
  await openBrowser('https://www.stickermule.com/claudecode');
};
```

## 核心内容

### 浏览器打开
```typescript
await openBrowser('https://www.stickermule.com/claudecode');
```

### 目标页面
- 地址：https://www.stickermule.com/claudecode
- 内容：Claude Code 贴纸订购
- 合作伙伴：Sticker Mule

## 设计点

1. **简单直接**：单一代码行完成功能
2. **品牌推广**：支持 Claude Code 周边
3. **外部合作**：与 Sticker Mule 合作
4. **异步处理**：使用 async/await

## 与其他文件的关系

- 导入 `openBrowser` 从 `../../utils/browser.js`
- 使用 `LocalCommandCall` 类型

## 注意事项

- 需要系统有默认浏览器
- 网络连接需要正常
- URL 可能变更
- 某些环境可能无法打开浏览器
