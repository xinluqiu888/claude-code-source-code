# clmTypes.ts — CLM允许类型定义

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/tools/PowerShellTool/clmTypes.ts`
- **作用**: PowerShell Constrained Language Mode允许的类型白名单

## 核心内容详解

### CLM_ALLOWED_TYPES

Microsoft CLM限制.NET类型使用到此允许列表。任何不在此集合中的类型被视为不安全。

```typescript
export const CLM_ALLOWED_TYPES: ReadonlySet<string> = new Set([
  // 类型加速器（短名称）
  'alias', 'array', 'bool', 'byte', 'char', 'datetime', 'decimal',
  'double', 'guid', 'hashtable', 'int', 'int16', 'int32', 'int64',
  'ipaddress', 'long', 'object', 'pscredential', 'pscustomobject',
  'regex', 'securestring', 'semver', 'string', 'switch', 'timespan',
  'uint', 'uri', 'version', 'void', 'xml',
  // ... 更多类型
  
  // SECURITY: 以下类型已被移除
  // - 'adsi', 'adsisearcher': 网络LDAP绑定
  // - 'wmi', 'wmiclass', 'wmisearcher': 远程WMI查询
  // - 'cimsession': CIM远程会话
])
```

### 类型名称规范化

```typescript
export function normalizeTypeName(name: string): string {
  return name
    .toLowerCase()
    .replace(/\[\]$/, '')      // 移除数组后缀: String[] → string
    .replace(/\[.*\]$/, '')    // 移除泛型参数: List[int] → list
    .trim()
}
```

### 类型检查

```typescript
export function isClmAllowedType(typeName: string): boolean {
  return CLM_ALLOWED_TYPES.has(normalizeTypeName(typeName))
}
```

## 安全说明

被移除的类型及其原因：

1. **ADSI/ADSISearcher**: 执行LDAP绑定，可能连接到恶意服务器
2. **WMI/WMIClass/WMISearcher**: 执行WMI查询，可访问远程主机和Win32_Process
3. **CIMSession**: 创建到远程主机的CIM会话

## 与其他文件的关系

- **powershellSecurity.ts**: 使用isClmAllowedType检查类型字面量
- **Microsoft文档**: https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_language_modes
