# keybindings/index.ts

## 文件描述
Keybindings 命令配置 - 打开键位配置文件

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local |
| 名称 | keybindings |
| 描述 | Open or create your keybindings configuration file |
| 启用条件 | isKeybindingCustomizationEnabled() |
| 支持非交互 | false |

## 核心内容

### 命令配置
```typescript
const keybindings = {
  name: 'keybindings',
  description: 'Open or create your keybindings configuration file',
  isEnabled: () => isKeybindingCustomizationEnabled(),
  supportsNonInteractive: false,
  type: 'local',
  load: () => import('./keybindings.js'),
} satisfies Command
```

## 设计点

1. **条件启用**：根据功能标志启用
2. **交互必需**：不支持非交互模式
3. **懒加载**：动态导入实现

## 与其他文件的关系

- 导入 `isKeybindingCustomizationEnabled` 从 `../../keybindings/loadUserBindings.js`

## 注意事项

- 打开键位配置文件
- 支持自定义键位
- 需要交互式环境
