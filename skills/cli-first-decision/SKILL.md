---
name: cli-first-decision
description: "优先使用 CLI 工具和子智能体委派的决策框架，任何任务前先判断最优工具路径；多步操作须合并/并行调用"
compatibility: "Requires OpenCode with Bash, task(), and CLI tools (fd, rg, jq, sd, curl)"
metadata:
  author: opencode
  version: "2.0"
---

# CLI-First Decision

**核心**：先选最短路径；能合并就合并；能并行就并行；闭合后 compress。

## 决策流程

1. 识别任务类型（修复 / 实现 / 调研 / 评估）
2. **合并判断**（见下节）— 多步是否可同轮/管道/并行
3. 单步简单（已知文件+内容）→ 内置工具 → compress
4. 多步/跨模块/生成/重构 → 拆 N 独立单元 → **一次全并行派发**
5. 机械操作 → 工具矩阵选 CLI（优先管道）
6. 有 skill → 先加载；有 MCP → 优先用
7. 架构/审查 → 自推或 Oracle

## 多步工具调用合并（硬规则）

**默认：同一响应里发出所有无依赖调用。禁止串行往返。**

| 类型 | 合并方式 | 何时 | 反例（禁止） |
|------|----------|------|--------------|
| 内置工具 | **同轮并行** | 多个 Read/Grep/Glob/lsp 互不依赖 | 先 Read A 再 Grep |
| CLI | **一条管道** | 查找→过滤→变换可串 | 三次 bash 分步 |
| 子智能体 | **一次派 N 个** | 独立单元 | 等 explore1 完再派 explore2 |
| 混合 | **同轮批出** | CLI + Read + task 无依赖 | 先 CLI 等完再 Read |

### 合并决策

```
需要 ≥2 次工具？
  ├─ 无依赖 → 同轮全部发出
  ├─ 可管道（A|B|C）→ 一条 CLI
  ├─ 有依赖 → 层内并行，层间串行
  └─ 仅 1 步 → 直接内置
```

## 并行规则

| 依赖 | 策略 |
|------|------|
| 无 | **全并行，同轮启动** |
| 有 | 层内并行，层间串行 |

### 推理/决策并行

**原则：事实收集可并行；互斥决策串行；独立判断轴同轮批出。**

```
推理任务？
  ├─ 需要事实？ → 同轮批出全部事实调用 → 再决策
  ├─ 多独立判断轴？ → 同轮并行 → 合成
  ├─ 互斥选项？ → 同轮按轴批评 → 选优
  ├─ 强因果单链？ → 一条 sequential-thinking
  └─ 已够决策 → 直接定
```

## 子智能体 Prompt（6 段必填）

```
1. TASK               原子目标
2. EXPECTED OUTCOME   交付物 + 成功标准
3. REQUIRED TOOLS     工具白名单
4. MUST DO            穷举需求
5. MUST NOT DO        禁止操作
6. CONTEXT            路径 + 模式 + 约束
```

**⚠️ WORKING_DIR 必填**：TASK 段后首行必须 `WORKING_DIR=<path>`。

**⚠️ `task_id` 陷阱**：`task_id` 是续聊(ses_...)不是新任务。新任务不传。

## Compress 时机

| 时机 | 动作 |
|------|------|
| 单步完成 | compress 该步 |
| 子智能体验证通过 | compress（文件名+签名+状态） |
| context 警告 | **立即** compress |

## 详细参考

工具矩阵、常见错误、红旗 → [references/REFERENCE.md](references/REFERENCE.md)
