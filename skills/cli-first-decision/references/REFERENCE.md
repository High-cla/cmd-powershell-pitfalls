# CLI-First Decision — Reference

Detailed tool matrix, error table, and common mistakes.

## 工具矩阵

| 场景 | skill / 工具 | CLI | 速度 |
|------|--------------|-----|------|
| 批量替换 | `git-sed` | `sd` | <1s |
| 文件查找 | `fd-find` | `fd` | <1s |
| 内容搜索 | `rg-search` | `rg` | <1s |
| AST 搜索替换 | `ast-grep` | `ast-grep` | <1s |
| JSON | `jq-query` | `jq` | <1s |
| YAML/XML | - | `yq` | <1s |
| HTTP | `git-curl` | `curl` | <1s |
| 模糊过滤 | `fzf-filter` | `fzf` | <1s |
| 代码统计 | - | `tokei` | <1s |
| 文本查看 | - | `bat` | <1s |
| 打包/换行 | `git-tar` / `git-dos2unix` | - | <1s |
| 子智能体 | `task()` | - | 数十秒 |
| 文件读写 | Read/Edit/Write | - | 即时 |
| 内容/文件搜索 | grep / glob | - | <1s |
| MCP 图谱 | codebase-memory-mcp | - | <5s |
| MCP 网页 | fetch | - | <10s |
| 推理/审查 | 自推 / Oracle | - | 视复杂度 |

CLI 来源: `~/AppData/Local/Microsoft/WinGet/Links/` + `~/.cargo/bin/`  
Alias: `find→fd`, `grep→rg`, `cat→bat`, `sed→sd`, `man→tldr`, `cd→z`

### 管道优先

| 目标 | 合并示例 | 勿拆成 |
|------|----------|--------|
| 找文件再搜内容 | `fd -e ts src \| xargs rg 'pattern'` | fd 轮 + rg 轮 |
| 搜再过滤 | `rg ... \| fzf --filter=...` | rg 再 fzf |
| JSON 变换 | `curl ... \| jq '.data[]'` | curl 存盘再 jq |
| 批量替换 | 一次 `sd`/`git-sed` 多文件 | 逐文件 Edit |
| 统计+列表 | `tokei` 或 `fd ... \| wc` | 多轮 ls/count |

### 内置工具同轮批

| 场景 | 同轮发出 |
|------|----------|
| 改前摸底 | 相关文件 Read 全部并行 |
| 定位符号 | Grep + Glob 并行 |
| 改后验收 | lsp_diagnostics 多文件并行 |
| 探索 | 多个 `task(explore/librarian, run_in_background=true)` 并行 |

## 场景速查

| 场景 | 做 | 禁 |
|------|----|----|
| 单文件已知修改 | 直接 Edit | 派子智能体 |
| 多文件摸底 | 同轮并行 Read/Grep | 串行逐个打开 |
| 查找+过滤+改 | 一条 CLI 管道 | 多轮 bash 接力 |
| 多步重构/生成 | 拆 + 同轮并行 task | 手动一步步 |
| 跨模块分析 | 多 explore 并行 | 自己逐个查 |
| 不熟库调研 | librarian | webfetch 硬啃 |
| 已知 bug | 一 task 或自推 | 忽视根因 |
| 多方案选型 | 同轮按轴批评 → 选优 | 串行深挖每个方案 |
| 正交审查 | 安全∥质量∥规格同轮并行 | 先审完一个再开下一个 |
| 决策缺事实 | 先同轮批事实再决策 | 边猜边调工具 |

## 常见错误

| # | 问题 | 修正 |
|---|------|------|
| 1 | 自己动手不委派 | 复杂语义 → 子智能体 |
| 2 | 串行不并行 | N 独立单元 → N 并行 |
| 3 | 多轮工具往返 | 无依赖 → 同轮全发出 |
| 4 | 可管道却拆轮 | A\|B\|C 一条命令 |
| 5 | 跳过 CLI 直接 task | 机械替换用 sd/git-sed |
| 6 | 找到文件后手改循环 | 一次批量替换 |
| 7 | 单文件也派代理 | 直接 Edit |
| 8 | CLI 不懂语义却硬 CLI | 语义任务用 task |
| 9 | sd 不 preview | 先 `--preview` |
| 10 | yq XML 参数混 | v4: `-p xml` |
| 11 | prompt 模糊 | 必须 6 段 |
| 12 | 不拆分复杂任务 | 一代理一事 |
| 13 | 闭合不 compress | 查 Compress 表 |
| 14 | 决策前串行摸底 | 事实调用同轮批出 |
| 15 | 多方案串行深挖 | 同轮按轴批评再选 |
| 16 | 决策后双轨实现 | 选定一条主路径 |
| 17 | task_id 误用于新任务 | 新任务不传 task_id |

## 红旗（立刻改）

- 本可并行的 Read/Grep 排成队列
- 三次独立 bash 本可 `|` 一条
- 等一个 explore 完再派下一个
- 「先看看再决定调什么」却只发了 1 个工具
- 多判断轴串成队列
- 事实未齐就开写，或决策未定双轨实现
- 新任务误传 `task_id`
