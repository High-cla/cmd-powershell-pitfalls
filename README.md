<p align="center">
  <img src="https://img.shields.io/badge/Windows-CMD%20%7C%20PowerShell-blue?style=flat-square&logo=windows&logoColor=white" />
  <img src="https://img.shields.io/badge/Language-ZH%20%7C%20EN-green?style=flat-square" />
  <img src="https://img.shields.io/github/license/High-cla/cmd-powershell-pitfalls?style=flat-square" />
</p>

<h1 align="center">CMD + PowerShell 混合脚本避坑指南</h1>
<p align="center"><i>Everything that can go wrong, will go wrong — when you mix CMD and PowerShell.</i></p>

---

> **中文**：编写 Windows 批处理（`.cmd` / `.bat`）并内嵌 PowerShell 命令时，常见的陷阱与解决方案。  
> **English**: A reference guide for common pitfalls when writing hybrid CMD (`.cmd` / `.bat`) scripts with embedded PowerShell commands.

---

## 📋 内容一览 | Overview

| #  | 类别 Category            | 问题 Pitfall                                            | 严重性 Severity |
| -- | ------------------------ | ------------------------------------------------------- | --------------- |
| 1  | 🔤 编码 Encoding         | `chcp 65001` 破坏 `%VAR%` 展开                            | 🔴 High         |
| 2  | 🔄 循环 Loops            | `for` 循环体内 `)` 被当作结束符吞掉                       | 🔴 High         |
| 3  | 📦 重定向 Redirection    | 括号 `()` 与输出重定向 `>` 的优先级问题                    | 🟡 Medium       |
| 4  | 📁 路径 Paths            | `%~dp0` 的正确使用与拼接方式                              | 🟢 Low          |
| 5  | ➖ 续行 Line Continuation | `^` 在双引号内不起作用                                    | 🟡 Medium       |
| 6  | ❝ 引号 Quoting          | `\"` 在 CMD 字符串中无效，应用单引号                       | 🔴 High         |
| 7  | 🧩 对象类型 Object Type  | `Get-Location` 返回 `PathInfo` 而非字符串                  | 🔴 High         |
| 8  | 💲 变量传递 Variable     | `$` / `%` 在 CMD → PowerShell 的传递规则                  | 🟡 Medium       |
| 9  | 🔍 过滤 Filtering        | `findstr` 正则语法有限                                    | 🟢 Low          |
| 10 | 🔗 管道 Pipeline         | `\|` 在 CMD 字符串中不会触发管道解析                      | 🟢 Low          |
| 11 | ⏳ 延迟展开 Delayed Exp. | 循环内改变量必须用 `!var!`                                | 🔴 High         |
| 12 | 📝 文件编码 File Encoding | `Set-Content` 在各 PowerShell 版本的默认编码差异          | 🟡 Medium       |
| 13 | ❌ 错误处理 Error        | `%ERRORLEVEL%` 与 `$LASTEXITCODE` 的正确使用              | 🟡 Medium       |
| 14 | ⚡ 性能 Performance      | `.NET` 直调 vs Cmdlet 的性能差距                           | 🟢 Low          |
| 15 | 🚪 退出码 Exit Code      | PowerShell 退出码需显式传递回 CMD                          | 🟡 Medium       |
| 16 | 📄 追加 vs 重定向        | 循环内 `>>` 逐个打开文件 vs 一次性 `>`                     | 🟢 Low          |
| 17 | ✅ 检查清单 Checklist    | 写完脚本后逐项自查                                        | —               |

---

## 🎯 适用场景 | When to Use This Guide

- 你正在编写 `.cmd` / `.bat` 批处理文件，需要内嵌 `powershell -Command "..."` 
- 你的脚本在同事电脑上跑不通，或者同样的代码有时行有时不行
- 你遇到了 `)` 不见了、变量展开乱码、`Get-Location` 报错等奇怪问题

---

## 🚀 快速示例 | Quick Example

```batch
@echo off
cd /d "%~dp0"

REM ✅ 用 .NET 提升性能
powershell -NoProfile -Command ^
  "$f='Steamtools.lua';" ^
  "$h=[IO.File]::ReadAllText($f);" ^
  "$r='(Steamtools\.lua|addappid\([^)]+\))';" ^
  "$m=[regex]::Matches($h,$r);" ^
  "if($m.Count) { [IO.File]::WriteAllText($f,$m.Value -join \"`r`n\",[Text.Encoding]::ASCII) }"
```

> 更多示例见 [`SKILL.md`](./SKILL.md)。

---

## 📖 完整文档 | Full Reference

完整 17 条规则及代码示例请阅读：

➡️ **[`SKILL.md`](./SKILL.md)**

---

## 🤝 贡献 | Contributing

发现新的坑？欢迎提 Issue 或 PR！

Found a new pitfall? Feel free to open an Issue or PR.

---

## 📄 许可 | License

MIT
