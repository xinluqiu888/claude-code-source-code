# file-index/index.ts — 纯TypeScript文件索引实现

## 基本信息

- **文件路径**: `/root/projects/claude-code-source-code/src/native-ts/file-index/index.ts`
- **类型**: TypeScript 模块
- **作用**: Rust NAPI模块（nucleo）的纯TypeScript移植，用于高性能模糊文件搜索

## 功能概述

本模块是对Rust NAPI模块vendor/file-index-src的纯TypeScript重新实现，包装了nucleo（由helix编辑器使用的模糊匹配库）提供相同的API和评分行为，但无需原生依赖。

## 核心内容详解

### API概览

```typescript
export class FileIndex {
  loadFromFileList(fileList: string[]): void                    // 同步加载
  loadFromFileListAsync(fileList: string[]): {                  // 异步加载
    queryable: Promise<void>  // 第一批索引完成即可查询
    done: Promise<void>       // 全部索引完成
  }
  search(query: string, limit: number): SearchResult[]         // 搜索文件
}

export type SearchResult = {
  path: string
  score: number  // 越低越好，0.0是最佳匹配
}
```

### 评分语义

- **分数计算**: `位置分数 = 结果位置 / 结果总数`，最佳匹配为0.0
- **测试文件惩罚**: 包含"test"的路径获得1.05x惩罚（上限1.0），使非测试文件排名略高

### nucleo风格评分常量

```typescript
const SCORE_MATCH = 16              // 匹配得分
const BONUS_BOUNDARY = 8            // 边界奖励（/ - _ . 空格）
const BONUS_CAMEL = 6               // 驼峰奖励
const BONUS_CONSECUTIVE = 4         // 连续匹配奖励
const BONUS_FIRST_CHAR = 8          // 首字符奖励
const PENALTY_GAP_START = 3         // 间隙开始惩罚
const PENALTY_GAP_EXTENSION = 1     // 间隙扩展惩罚
```

### 核心算法

**预计算优化**：
1. **位图过滤**: 每个路径预计算a-z字符位图，O(1)快速排除不含查询字符的路径
2. **小写缓存**: 预计算小写路径用于大小写不敏感匹配
3. **长度缓存**: 预缓存路径长度

**搜索流程**：
1. **位图预过滤**: `(charBits[i] & needleBitmap) !== needleBitmap`
2. **贪婪最早位置**: 使用`indexOf`找到最早的匹配位置
3. **间隙计算**: 记录字符间间隙惩罚和连续匹配奖励
4. **边界评分**: 检查匹配位置前的字符（/ - _ . 空格或驼峰）
5. **Top-K优化**: 维护排序的best `limit`匹配，避免全量排序

### 异步加载

```typescript
loadFromFileListAsync(fileList: string[]): { queryable: Promise<void>, done: Promise<void> }
```

- 每4ms让出事件循环（CHUNK_MS常量）
- 第一批索引完成后即可查询（返回部分结果）
- 适用于270k+文件的大索引

### Top-Level缓存

提取唯一的顶级路径段，按（长度升序，字母升序）排序：
- 处理Unix（/）和Windows（\）路径分隔符
- 限制100个缓存项
- 空查询时返回此缓存

## 设计要点

1. **智能大小写**: 查询全小写→大小写不敏感；含大写→大小写敏感
2. **SIMD加速**: `indexOf`在JSC/V8中使用SIMD加速
3. **内存优化**: 重用Int32Array位置缓冲区，避免重复分配
4. **NaN安全相等**: `a === b || (a !== a && b !== b)`处理NaN比较

## 与其他文件的关系

- **替代 native module**: 取代vendor/file-index-src的Rust实现
- **被搜索功能使用**: 用于文件快速搜索和补全

## 注意事项

1. **最大查询长度**: 64字符（MAX_QUERY_LEN），超过会被截断
2. **位图限制**: 只索引a-z字符，非ASCII字符依赖边界/连续匹配评分
3. **路径分隔符**: 统一处理/和\两种分隔符
4. **测试导出**: `__test`对象导出内部函数供测试使用
