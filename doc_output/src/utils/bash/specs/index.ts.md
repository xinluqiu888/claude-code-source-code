# index.ts — bash specs集合导出

> **一句话总结**：收集并导出所有内置命令规格（specs）的索引文件。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/specs/index.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~19行 |
| 主要职责 | 内置命令规格的集中导出 |

---

## 功能概述

该文件是所有内置CommandSpec的入口点，从各个spec文件导入并导出为统一数组。

包含的specs：
- pyright - Python类型检查器
- timeout - 命令超时执行
- sleep - 延迟执行
- alias - 别名管理
- nohup - 后台执行
- time - 计时执行
- srun - SLURM集群执行

---

## 与其他文件的关系

- **依赖**：
  - 所有同级spec文件
- **被依赖**：
  - `../registry.ts` - 加载内置specs
