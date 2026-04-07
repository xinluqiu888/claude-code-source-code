# index.ts — 设置同步服务

## 基本信息

- **路径**: `/root/projects/claude-code-source-code/src/services/settingsSync/index.ts`
- **作用域**: 用户设置和记忆文件跨环境同步
- **主要导出**:
  - `uploadUserSettingsInBackground`: 后台上传用户设置
  - `downloadUserSettings`: 下载用户设置（启动时）
  - `redownloadUserSettings`: 强制重新下载（`/reload-plugins`）

## 功能概述

同步用户设置和记忆文件跨 Claude Code 环境：
- **交互式 CLI**: 上传本地设置到远程（增量，仅更改条目）
- **CCR**: 下载远程设置到本地（插件安装前）

后端 API: anthropic/anthropic#218817

## 核心内容详解

### 常量配置

```typescript
const SETTINGS_SYNC_TIMEOUT_MS = 10000      // 10 秒
const DEFAULT_MAX_RETRIES = 3
const MAX_FILE_SIZE_BYTES = 500 * 1024      // 500 KB（后端限制）
```

### 上传功能

#### `uploadUserSettingsInBackground()`
后台上传用户设置（交互式 CLI）：

**条件检查**:
- `UPLOAD_USER_SETTINGS` 特性开启
- `tengu_enable_settings_sync_push` 特性标志
- 交互式会话
- 使用 OAuth

**流程**:
1. 获取远程设置
2. 构建本地条目
3. 对比找出变更条目
4. 上传变更
5. 记录遥测

**最佳努力**: 失败不阻塞启动

### 下载功能

#### `downloadUserSettings()`
下载用户设置（CCR 模式）：

**缓存 Promise**: 启动时首次调用开始获取，后续调用加入同一 Promise

**条件检查**:
- `DOWNLOAD_USER_SETTINGS` 特性开启
- `tengu_strap_foyer` 特性标志
- 使用 OAuth

**流程**:
1. 获取远程设置
2. 应用远程条目到本地文件
3. 记录遥测
4. 返回是否应用了设置

#### `redownloadUserSettings()`
强制重新下载（`/reload-plugins` 使用）：
- 绕过缓存 Promise
- 无重试（用户主动命令）
- 调用者负责触发 `settingsChangeDetector.notifyChange`

### 核心辅助函数

#### `fetchUserSettings(maxRetries?)`
带重试的设置获取：
- 认证头部构建
- OAuth token 刷新
- 指数退避重试
- 404 表示空数据

#### `uploadUserSettings(entries)`
设置上传：
- PUT 请求
- 返回校验和和时间戳

#### `buildEntriesFromLocalFiles(projectId)`
从本地文件构建同步条目：

扫描的文件：
1. 全局用户设置（`~/.claude/settings.json`）
2. 全局用户记忆（`~/.claude/CLAUDE.md`）
3. 项目本地设置（`.claude/settings.local.json`）
4. 项目本地记忆（`CLAUDE.local.md`）

过滤条件：
- 文件不存在 -> 跳过
- 空/仅空白 -> 跳过
- 超过 500KB -> 跳过

#### `applyRemoteEntriesToLocal(entries, projectId)`
应用远程条目到本地文件：

**安全检查**:
- 仅匹配预期键
- 大小限制检查（防御性）

**缓存失效**:
- 设置文件写入后调用 `resetSettingsCache()`
- 记忆文件写入后调用 `clearMemoryFileCaches()`

**内部写入标记**:
- 使用 `markInternalWrite` 防止虚假变更检测

### OAuth 检查

#### `isUsingOAuth()`
检查用户是否使用第一方 OAuth：
- 第一方 provider
- 第一方 base URL
- 有访问 token
- 包含 `user:inference` scope

**注意**: 仅检查 `user:inference`，CCR 的文件描述符 token 硬编码此 scope

### 文件操作

#### `tryReadFileForSync(filePath)`
带大小限制的文件读取：
- 返回 null：不存在、空、超过 500KB

#### `writeFileForSync(filePath, content)`
同步文件写入：
- 自动创建父目录
- UTF-8 编码

## 设计要点

1. **增量同步**: 仅上传变更条目
2. **后台执行**: 上传不阻塞启动
3. **Promise 缓存**: 启动下载缓存避免重复请求
4. **大小限制**: 500KB 每文件上限
5. **缓存失效**: 写入后及时失效相关缓存
6. **内部写入标记**: 防止同步触发自身检测
7. **OAuth 限制**: 仅 OAuth 用户支持同步

## 与其他文件的关系

- **types.ts**: 类型定义和 Zod Schema
- **auth.ts**: OAuth token 管理
- **settingsCache.ts**: 设置缓存管理
- **claudemd.ts**: 记忆文件缓存管理

## 注意事项

1. **Feature Flags**:
   - `UPLOAD_USER_SETTINGS`: 上传功能开关
   - `DOWNLOAD_USER_SETTINGS`: 下载功能开关
   - `tengu_enable_settings_sync_push`: 上传特性标志
   - `tengu_strap_foyer`: 下载特性标志

2. **同步文件**: 仅同步 4 个特定文件
3. **项目 ID**: 从 git remote 哈希获取
4. **失败开放**: 所有操作失败不阻塞主流程
5. **无重试重新下载**: `redownloadUserSettings` 无重试
