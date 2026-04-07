# powershellPermissions.ts — PowerShell权限主入口

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/PowerShellTool/powershellPermissions.ts`
- **作用**: PowerShell权限检查的主入口点

## 核心内容详解

### 权限检查流程

```typescript
export async function checkPowerShellPermissions(
  input: { command: string; timeout?: number },
  context: ToolPermissionContext,
): Promise<PermissionDecision>
```

#### 步骤

1. **解析命令**: 使用PowerShell原生解析器
2. **检查bypass/dontAsk模式**: 直接通过
3. **安全检查** (powershellSecurity.ts):
   - Invoke-Expression
   - 编码命令
   - 下载cradle
   - COM对象
   - 等等
4. **路径约束** (pathValidation.ts): 验证路径安全
5. **权限模式** (modeValidation.ts): acceptEdits模式特殊处理
6. **只读验证** (readOnlyValidation.ts): 检查是否为只读命令

### 决策结果

- **allow**: 自动允许
- **ask**: 请求用户确认（带警告信息）
- **deny**: 拒绝执行
- **passthrough**: 传递到下一个检查器

## 设计要点

1. **分层验证**: 多个专门的验证器协同工作
2. **短路返回**: 任一验证器要求ask时立即返回
3. **详细消息**: 提供具体的拒绝原因

## 与其他文件的关系

- **powershellSecurity.ts**: AST级安全检查
- **pathValidation.ts**: 路径验证
- **modeValidation.ts**: 权限模式验证
- **readOnlyValidation.ts**: 只读命令验证
