# srun.ts — SLURM srun命令规格定义

> **一句话总结**：定义SLURM集群srun命令的CommandSpec，支持集群任务调度参数。

---

## 基本信息

| 项目 | 内容 |
|------|------|
| 文件路径 | `src/utils/bash/specs/srun.ts` |
| 文件类型 | TypeScript |
| 代码行数 | ~32行 |
| 主要职责 | SLURM srun命令的规格定义 |

---

## 功能概述

srun是SLURM（Simple Linux Utility for Resource Management）的作业启动命令，用于在集群节点上运行任务。

---

## 核心内容详解

### CommandSpec定义

- **name**: `'srun'`
- **description**: `'Run a command on SLURM cluster nodes'`

### Options

| 选项 | 描述 | 参数 |
|------|------|------|
| `-n, --ntasks` | 任务数量 | count |
| `-N, --nodes` | 节点数量 | count |

### Args

- **name**: 'command'
- **description**: 'Command to run on the cluster'
- **isCommand**: true - 标记为包装命令

### isCommand标记

srun会包装实际执行的命令，权限系统会递归提取内部命令。

---

## 与其他文件的关系

- **被依赖**：
  - `./index.ts` - 导出到specs集合
