# index.ts — doctor 命令入口配置

## 基本信息

| 属性 | 值 |
|------|-----|
| 文件路径 | `/root/projects/claude-code-source-code/src/commands/doctor/index.ts` |
| 文件类型 | TypeScript (.ts) |
| 行数 | 12 行 |
| 主要职责 | 定义 doctor 命令的元数据和启用条件 |

## 核心内容详解

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

### 关键特性

| 属性 | 值 | 说明 |
|------|-----|------|
| `name` | `'doctor'` | 命令名 |
| `description` | 诊断和验证安装设置 | 功能描述 |
| `isEnabled` | 函数 | 检查 DISABLE_DOCTOR_COMMAND |
| `type` | `'local-jsx'` | 本地 JSX 组件 |

### 环境变量控制

可通过 `DISABLE_DOCTOR_COMMAND` 环境变量禁用该命令：
```bash
DISABLE_DOCTOR_COMMAND=1 claude
```

## 设计要点

1. **可禁用**：通过环境变量提供禁用选项
2. **诊断功能**：帮助用户排查安装和配置问题
3. **默认启用**：无特殊环境变量时默认启用

## 与其他文件的关系

- **doctor.tsx**: 实际的诊断界面实现
- **envUtils.ts**: 提供 `isEnvTruthy` 函数
