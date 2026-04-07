# chrome.tsx — Claude in Chrome 设置界面

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/chrome/chrome.tsx` |
| 文件类型 | TypeScript React (.tsx) |
| 主要职责 | 提供 Claude in Chrome 浏览器扩展的设置和管理界面 |

## 功能概述

该文件实现了 `/chrome` 命令的UI组件，用于配置和管理 Claude in Chrome 功能。这是一个功能丰富的设置界面，允许用户安装/重新连接 Chrome 扩展、管理权限、以及设置默认启用状态。

## 核心内容详解

### 主要组件

**ClaudeInChromeMenu 组件**
- 显示当前连接状态（已连接/已断开）
- 显示扩展安装状态（已安装/未检测到）
- 提供操作选项菜单

### 支持的操作

| 操作 | 功能 |
|------|------|
| `install-extension` | 安装 Chrome 扩展 |
| `reconnect` | 重新连接扩展 |
| `manage-permissions` | 管理站点权限 |
| `toggle-default` | 切换默认启用状态 |

### 常量定义

```typescript
const CHROME_EXTENSION_URL = 'https://claude.ai/chrome';
const CHROME_PERMISSIONS_URL = 'https://clau.de/chrome/permissions';
const CHROME_RECONNECT_URL = 'https://clau.de/chrome/reconnect';
```

## 设计要点

1. **平台检测**：检测是否在 WSL 环境中（暂不支持）
2. **订阅检查**：检查用户是否为 Claude.ai 订阅者
3. **状态显示**：清晰显示扩展和连接状态
4. **链接打开**：支持在浏览器或 Chrome 中打开相关页面

## 与其他文件的关系

- **claudeInChrome/setup.ts**: 提供扩展安装检测
- **config.ts**: 全局配置读写
- **browser.ts**: 浏览器打开功能
- **index.ts**: 命令注册
