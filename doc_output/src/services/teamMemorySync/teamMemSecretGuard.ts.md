# teamMemSecretGuard.ts — 团队记忆密钥守卫

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/teamMemorySync/teamMemSecretGuard.ts`
- **作用域**: 阻止密钥写入团队记忆文件
- **主要导出**:
  - `checkTeamMemSecrets`: 检查团队记忆文件是否包含密钥

## 功能概述

检查写入/编辑团队记忆路径的文件是否包含密钥。在 `FileWriteTool` 和 `FileEditTool` 的 `validateInput` 中调用，防止模型将密钥写入团队记忆文件（这会同步给所有仓库协作者）。

## 核心内容详解

### 核心函数

#### `checkTeamMemSecrets(filePath, content)`
检查文件内容是否包含密钥：

**参数**:
- `filePath`: 文件路径
- `content`: 文件内容

**返回**:
- `string | null`: 如果包含密钥，返回错误消息；否则返回 `null`

**逻辑**:
1. 检查 `TEAMMEM` 特性是否开启
2. 动态导入 `teamMemPaths.js` 和 `secretScanner.js`
3. 检查路径是否为团队记忆路径
4. 扫描内容中的密钥
5. 返回错误消息（如果找到密钥）

**错误消息**:
```
Content contains potential secrets ({labels}) and cannot be written to team memory. 
Team memory is shared with all repository collaborators. 
Remove the sensitive content and try again.
```

### 特性保护

```typescript
if (feature('TEAMMEM')) {
  /* eslint-disable @typescript-eslint/no-require-imports */
  const { isTeamMemPath } = require('../../memdir/teamMemPaths.js')
  const { scanForSecrets } = require('./secretScanner.js')
  /* eslint-enable @typescript-eslint/no-require-imports */
  // ...
}
```

使用特性门控保持在外部构建中不活跃（`TEAMMEM` 特性关闭时）。

### 动态导入

使用 `require` 动态导入：
1. `teamMemPaths.isTeamMemPath`: 检查路径是否为团队记忆路径
2. `secretScanner.scanForSecrets`: 扫描内容中的密钥

动态导入确保：
- 仅在 `TEAMMEM` 特性开启时加载
- 避免循环依赖
- 外部构建中代码被消除

## 设计要点

1. **特性门控**: 内部检查 `feature('TEAMMEM')`，调用者可无条件导入
2. **动态导入**: 延迟加载依赖
3. **路径检查**: 仅检查团队记忆路径
4. **密钥扫描**: 使用 `scanForSecrets` 检测密钥
5. **用户友好**: 清晰的错误消息说明为什么不能写入

## 与其他文件的关系

- **secretScanner.ts**: 提供 `scanForSecrets`
- **teamMemPaths.ts**: 提供 `isTeamMemPath`
- **FileWriteTool/FileEditTool**: 调用此函数

## 注意事项

1. **外部构建**: `TEAMMEM` 为 false 时，函数始终返回 `null`
2. **运行时组装**: `secretScanner` 运行时组装敏感前缀
3. **仅检查**: 不修改内容，仅检查
