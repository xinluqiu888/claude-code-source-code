# PluginHintMenu.tsx

组件用于显示插件安装提示对话框，提供用户选择是否安装推荐插件的交互界面。

## 基本信息

- **文件路径**: `src/components/ClaudeCodeHint/PluginHintMenu.tsx`
- **组件名称**: PluginHintMenu
- **类型**: React 函数组件

## 功能概述

该组件在终端中显示一个插件推荐对话框，当用户执行某个命令时，如果该命令建议安装特定插件，此组件会弹出提示让用户选择是否安装。

## 核心内容

### Props 接口

| 属性 | 类型 | 说明 |
|------|------|------|
| pluginName | string | 插件名称 |
| pluginDescription | string | 插件描述（可选） |
| marketplaceName | string | 应用市场名称 |
| sourceCommand | string | 触发提示的源命令 |
| onResponse | function | 用户响应回调，返回 'yes'、'no' 或 'disable' |

### 主要常量

- **AUTO_DISMISS_MS**: 30000 毫秒，30秒后自动关闭对话框并返回'no'

### 响应选项

1. **Yes, install {pluginName}** - 确认安装插件
2. **No** - 取消安装
3. **No, and don't show plugin installation hints again** - 取消安装并禁用未来提示

### 自动关闭机制

使用 `useEffect` 和 `useRef` 设置30秒自动关闭定时器，防止对话框长时间占用界面。

## 设计要点

- 使用 `PermissionDialog` 作为外层容器，提供统一的权限对话框样式
- 使用 `Select` 组件实现选项选择
- 支持按 ESC 键取消（触发 onCancel 回调）
- 使用 Ink 组件（Box、Text）进行终端布局

## 与其他文件的关系

- **引入组件**:
  - `../ink.js` - Ink UI 组件库（Box、Text）
  - `../CustomSelect/select.js` - 自定义选择组件
  - `../permissions/PermissionDialog.js` - 权限对话框容器

## 备注

- 组件通过 ref 保存回调函数以确保定时器中的引用始终是最新的
- 用户选择 "disable" 选项后，应在应用层持久化设置以禁用未来提示
