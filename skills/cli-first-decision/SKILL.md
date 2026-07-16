---
name: cli-first-decision
description: "优先使用 CLI 工具和子智能体委派的决策框架，任何任务前先判断最优工具路径"
---

# CLI-First Decision

## 优先级决策链

```
触发 -> 思考 -> 拆分子智能体 -> 内置工具 -> CLI -> Skill -> MCP -> 自行推理 -> 压缩
```

## 决策树

```
收到请求
  |- 识别类型：修复 / 实现 / 调研 / 评估？
  |    |- 先理解问题本质，别急着摸工具
  |    |- 单步简单操作？
  |    |   (已知文件 + 已知内容)
  |    |   -> 直接内置工具，完后 compress
  |    |- 多步 / 跨模块 / 代码生成 / 重构 / 调研？
  |    |   |- 拆成 N 个独立单元
  |    |   |- 无依赖 -> 全并行
  |    |   |- 有依赖 -> 层内并行，层间串行
  |    |   |- 每个 prompt 必须 6 部分
  |    |   -> task(run_in_background=true) 并行派发
  |    |- 机械操作？
  |    |   |- 批量替换 -> git-sed / sd
  |    |   |- 文件查找 -> fd-find / glob
  |    |   |- 内容搜索 -> rg-search / grep
  |    |   |- AST 操作 -> ast-grep
  |    |   |- JSON/YAML -> jq-query / yq
  |    |- 有领域 skill？-> 先加载再执行
  |    |- 有 MCP 工具？
  |    |   |- 代码关系 -> codebase-memory-mcp
  |    |   |- 顺序推理 -> sequential-thinking
  |    |   |- 网页抓取 -> fetch
  |    -> 架构/设计/审查？-> 自行推理 or Oracle
```

## 压缩四阶段

```
1. 主智能体干完活 -> compress (即使没派子智能体)
2. 派发子智能体后 -> compress (不保留规划细节)
3. 子智能体结果验证通过 -> compress (只保留文件名+签名+状态)
4. 全部完成集成验证后 -> compress (保留目标+产出+关键决策)
```

## 子智能体优先

**Default: DELEGATE**

- 凡是可以委派的，一律委派
- N 个独立逻辑单元 -> N 个并行 `run_in_background=true`
- 子智能体内可自由调工具 (内置/CLI/MCP/LSP/web)
- prompt 必须含 6 部分:

```
1. TASK          原子目标，一个 delegation 做一件事
2. EXPECTED OUTCOME  交付物 + 成功标准
3. REQUIRED TOOLS   工具白名单
4. MUST DO       穷举需求
5. MUST NOT DO   禁止操作
6. CONTEXT       文件路径 + 模式 + 约束
```

- 单步简单操作例外 (已知文件 + 已知内容 + 单文件) -> 直接内置工具

## 并行派发模式

```
// 正确: N 个独立单元并行
task(run_in_background=true, prompt="单元A...")
task(run_in_background=true, prompt="单元B...")
task(run_in_background=true, prompt="单元C...")
compress -> 结束响应，等通知

// 错误: 串行等待
result1 = task(prompt="单元A...")
result2 = task(prompt="单元B...")
```

## 复杂任务拆分方法论

### 拆分原则

```
复杂任务
  |- 1. 识别天然边界 (模块/功能/层级/关注点)
  |- 2. 验证独立性 (依赖否？改同文件否？)
  |- 3. 确定粒度 (过粗质量差，过细开销大)
        每个子智能体 = 1 个逻辑闭合的可验证交付物
```

### 5 种拆分模式

| 模式 | 场景 | 做法 | 并行度 |
|------|------|------|--------|
| 水平 | 同层多独立模块 | 路由/组件各一个 | 全并行 |
| 垂直 | 分层架构 | 每层一个，定义数据契约 | 流水线 |
| 功能 | 多功能特性 | 每个功能一个 | 全并行 |
| 阶段 | 分析->设计->实现 | 先调研再异步实现 | 调研先行 |
| 角度 | 同一模块多维度 | 实现+测试+文档各一个 | 全并行 |

### 依赖处理

```
无依赖   -> 全并行 (同时启动，同时收)
有依赖   -> 分层并行 (层内并行，层间串行)
部分依赖 -> 谱系并行 (核心先完成->依赖者再并行)
```

### 结果验证

```
收集 -> 每个 task 返回后:
  |- 交付物存在？
  |- lsp_diagnostics 干净？
  |- 符合 MUST DO？
  -> 通过后 compress

全部完成后:
  |- 编译/类型检查通过？
  |- 测试通过？
  -> 最终 compress
```

## 场景决策速查

| 场景 | 该做什么 | 别做什么 |
|------|---------|---------|
| 单文件已知内容修改 | 直接 Edit | 别派子智能体 |
| 多步重构/代码生成 | 拆 + 并行子智能体 | 别手动一步步 |
| 跨模块分析 | 派 explore 并行 | 别自己一个个查 |
| 新功能实现 | 拆 + 并行子智能体 | 别自己写所有代码 |
| 调研不熟库 | 派 librarian | 别 webfetch 硬读 |
| 修复已知 bug | 派一个子智能体或自推 | 别忽视 |

## 工具矩阵

| 层级 | 场景 | 工具 (Skill) | CLI 直接命令 | 耗时 |
|------|------|------|------|------|
| 子智能体 | 多步推理/代码生成/调研 | `task()` | - | 数十秒 |
| 内置 | 文件读写编辑 | Read/Edit/Write | - | 即时 |
| 内置 | 内容搜索 | grep | - | <1s |
| 内置 | 文件匹配 | glob | - | <1s |
| 内置 | 文件内容分析 | look_at | - | <1s |
| CLI | 批量替换 | git-sed | `sd` (alias: sed) | <1s |
| CLI | 文件查找 | fd-find | `fd` (alias: find) | <1s |
| CLI | 内容搜索 | rg-search | `rg` (alias: grep) | <1s |
| CLI | AST 操作 | ast-grep | `ast-grep` | <1s |
| CLI | JSON 查询 | jq-query | `jq` | <1s |
| CLI | XML/YAML | - | `yq` | <1s |
| CLI | HTTP 请求 | git-curl | `curl` | <1s |
| CLI | 模糊过滤 | fzf-filter | `fzf` | <1s |
| CLI | 打包解压 | git-tar | - | <1s |
| CLI | 换行符转换 | git-dos2unix | - | <1s |
| CLI | 代码统计 | - | `tokei` | <1s |
| CLI | 文本查看 | - | `bat` (alias: cat) | <1s |
| CLI | 字段提取 | - | `choose` | <1s |
| CLI | 简版 man | - | `tldr` (alias: man) | <1s |
| CLI | 智能跳转 | - | `z` (zoxide, alias: cd) | <1s |
| Skill | 领域专知 | 加载 skill | - | 文档 |
| MCP | 知识图谱 | codebase-memory-mcp | - | <5s |
| MCP | 顺序推理 | sequential-thinking | - | 步数 |
| MCP | 网页 | fetch | - | <10s |
| 推理 | 架构/审查 | 自推 or Oracle | - | 视复杂度 |

> CLI 直接命令: 位于 `~/AppData/Local/Microsoft/WinGet/Links/` (WinGet) 和 `~/.cargo/bin/` (cargo)
> Shell alias 通过 `~/.bashrc` + `BASH_ENV` 环境变量全局生效

## CLI 速查

### Skill 命令 (通过 OpenCode 调用)

```
git-sed      表达式批量替换    git-sed expression:"s/old/new/g" files:["*.ts"] inPlace:""
fd-find      快速文件查找      fd-find pattern:"*.xml" extension:"xml" path:"./"
rg-search    高速内容搜索      rg-search pattern:"foo" glob:"*.ts" context:2
ast-grep     AST 搜索/替换     ast_grep_search pattern:"if($$$){}" lang:"ts"
jq-query     JSON 查询         jq-query filter:".name" file:"data.json"
git-curl     HTTP API 请求     git-curl url:"https://api.example.com" method:"POST"
fzf-filter   模糊过滤          fzf-filter pattern:"keyword" input:$lines
git-tar      打包解压          git-tar action:"create" archive:"out.tar.gz" files:["src/"]
git-dos2unix 换行符转换        git-dos2unix file:"script.sh" mode:"dos2unix"
```

### CLI 直接命令 (PATH 中全局可用)

```
sd           简易查找替换      sd find:"old" replace:"new" files:["file.txt"]
fd           文件查找          fd pattern:"*.xml" --extension xml
rg           内容搜索          rg "pattern" --glob "*.ts"
ast-grep     AST 搜索/替换     ast-grep search -p "if($$$){}" -l ts
jq           JSON 查询         jq '.name' data.json
yq           YAML/XML 查询     yq '.root.item' data.xml -p xml
fzf          模糊过滤          cat file | fzf -q "keyword"
bat          文本查看          bat file.ts
choose       字段提取          echo "a,b,c" | choose 0
tokei        代码统计          tokei path:"./"
tldr         简版 man          tldr fd
z            智能跳转          z project-name
curl         HTTP 请求         curl -s https://api.example.com
grep         基础搜索 (内置)   grep "foo" *.ts
glob         文件匹配 (内置)   glob pattern:"**/*.ts"
```

> Shell alias: `find -> fd`, `grep -> rg`, `cat -> bat`, `sed -> sd`, `man -> tldr`, `cd -> z`

## Session Continuation

子智能体返回 `ses_...` ID。后续跟进用 `task(task_id="ses_...")`，**不要重新创建**。

```
正确: task(task_id="ses_abc123", prompt="修复: 42行类型错误")
错误: task(category="quick", prompt="修复类型错误...")
```

## 常见错误

```
#1  自己动手不委派       -> 子智能体比你做得好
#2  串行不并行           -> 4 个独立单元就 4 个并行
#3  跳过 CLI 直接 task   -> git-sed 一秒搞定批量替换
#4  找到文件后手动改      -> git-sed 一行搞定搜索+替换
#5  单文件修改也用代理    -> 直接 Edit，别绕弯
#6  该用子智能体却硬扛    -> CLI 不懂语义，子智能体懂
#7  sd 不 preview        -> sd 默认直接改文件，先 --preview
#8  yq XML 参数版本混淆   -> v4 用 -p xml 不是 --xml
#9  prompt 太模糊        -> <5 行就是太模糊，必须 6 部分
#10 复杂任务不拆分       -> 别让一个智能体做全局事
#11 派发了不压缩          -> 上下文爆炸，派发完就 compress
```
