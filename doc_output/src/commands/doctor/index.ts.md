# doctor/index.ts

## 文件描述
Doctor 命令配置 - 诊断安装和设置

## 基本信息

| 属性 | 值 |
|------|-----|
| 类型 | local-jsx |
| 名称 | doctor |
| 描述 | Diagnose and verify your Claude Code installation and settings |
| 启用条件 | !DISABLE_DOCTOR_COMMAND |

## 核心内容

### 命令配置
```typescript
const doctor: Command = {
  name: 'doctor',
  description: 'Diagnose and verify your Claude Code installation and settings',
  isEnabled: () => !isEnvTruthy(process.env.DISABLE_DOCTOR_COMMAND),
  type: 'local-jsx',
  load: () => import('./doctor.js'),
}
```

## 设计点

1. **条件启用**：可通过环境变量禁用
2. **诊断功能**：检查安装和配置
3. **懒加载**：动态导入组件

## 与其他文件的关系

- 导入 `isEnvTruthy` 从 `../../utils/envUtils.js`

## 注意事项

- 检查系统配置
- 验证依赖项
- 提供修复建议
