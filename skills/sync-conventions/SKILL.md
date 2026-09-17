---
name: sync-conventions
description: 检测并同步跨项目的标准约定（AGENTS.md / docs/ROADMAP.md / docs/DECISIONS.md 等）在 project-scaffold 模板与各活跃项目之间的漂移。当用户说「同步约定」「检查约定漂移」「scaffold 漂移 / doctor」，或刚改完 project-scaffold/_template 想让已有项目跟进时使用。
---

# 同步跨项目约定

## 背景：为什么这不是一个脚本能解决的事

`~/Documents/projects/project-scaffold/_template/` 是各项目约定文件的**唯一真相源**。
但 **git 的版本单位是「仓库」，而约定文件属于「一族仓库」**——模板更新后，
已有项目不会自动跟进，于是产生**漂移**。

在这之前，漂移只是「不够整齐」；在 LLM 时代它变成了**正确性问题**：
**agent 会把过期的约定文件当真理执行**，而且不会报错。

正因如此，这件事天然分成两半：

| | 谁做 | 做什么 |
|---|---|---|
| **确定性** | `scaffold-doctor` 脚本 | 检测：哪条约定在哪个项目缺失 |
| **语义** | **你（agent）** | 判断：该同步？该适配？还是该项目合理的本地定制？ |

**不要把模板文字机械复制过去。** 同一条约定在不同项目的正确表述本就不同——
实测中「验证需给出结论」这条，正文项目写成「没做完」，笔记项目写成「没写完」，
**正则都得容忍这种差异，你更不该抹平它**。

## 步骤

### 1. 检测（确定性）

```bash
~/Documents/projects/project-scaffold/scaffold-doctor --json
```

读 `summary` 与每个 `projects[].drift`。约定本身带 `kind` 字段：

- **`structural`** — 结构性：补标题 / 补一条规则即可，**你能独立完成**
- **`content`** — 内容性：需要**人提供素材**（例如「考虑过的其他选项」必须真有其事）。
  **你只能搭骨架 + 列出待补清单，不要编造内容。**

### 2. 逐条判断（语义 —— 这一步是你的核心价值）

对每个漂移项，**先读该项目的上下文再决定**，不要只看检测结果：

1. 读该项目的 `AGENTS.md`（技术栈 / 命令 / 红线）
2. 读该项目的 `docs/DECISIONS.md`（已定决策，避免冲突）
3. 读该项目的 `docs/ROADMAP.md`（当前状态）

三种处理，必须显式选一个：

| 处理 | 何时用 |
|---|---|
| **同步** | 约定适用，项目只是没跟上 |
| **适配后同步** | 约定适用，但要改写成该项目的具体措辞 / 命令 / 验证方式 |
| **跳过，反向改模板** | ① 该项目其实已用别的方式满足（**检测器的误报**）② 该项目的写法更好 ③ 该约定对它不适用 |

> ⚠️ **检测结果只是信号，不是判决。** 正则匹配的是**结构**，不是**意图**。
> 实测两种情况：
> - 「验证需给出结论」这条，`quant-notes` 写的是「没写完」而非「没做完」——
>   精确匹配会把它**误报**为缺失（本仓库用正则容忍了这种差异）；
> - 「ROADMAP 有下一步小节」对 `webpokerdealer` 报了结构缺失，但读文件才发现
>   **内容其实写在「当前状态」的散文里**——是真漂移，只是解法应该是
>   **搬到独立小节**，而不是照抄模板。
>
> **遇到不符直觉的检测结果，先读文件确认**，再决定同步、还是放宽 `conventions.toml` 的正则。

### 3. 落地

- **一次只改一个项目**，改完立刻验证，再动下一个。
- 遵守**该项目自己的**提交信息规范（读它的 `AGENTS.md`）。
- 结构性约定：把标题 / 规则加上，并**按该项目语境改写正文**。
- 内容性约定：只加骨架（标题 + 表格 + 一行 `TODO: 待补`），**不编造**。

### 4. 验证（必须给出明确结论）

```bash
# ① 整体漂移是否收敛
~/Documents/projects/project-scaffold/scaffold-doctor

# ② 在该项目目录下跑它自己的测试（文档改动也要跑——很多项目把文档一致性做成了测试）
cd <项目> && uv run pytest        # 或 uv run python manage.py test
```

**没有验证 = 没做完。** 不要用「应该没问题」结案。

### 5. 报告

按项目汇总，四列缺一不可：

| 项目 | 约定 | 处理 | 依据 |
|---|---|---|---|
| exam-sim | roadmap-next-step | 同步 | 确实没有独立小节，接手者要翻散文才知道从哪继续 |
| webpokerdealer | roadmap-next-step | 跳过（误报） | 内容已在「当前状态」里，无需重复 |

再附上第 4 步的**验证结论**（测试数量、doctor 收敛情况）。

## 维护约定登记表

新增一条约定 = 两处改动：

1. 在 `project-scaffold/_template/` 的对应文件里把它写清楚（**这才是真相源**）
2. 在 `project-scaffold/conventions.toml` 登记一条 `pattern`

`pattern` 用**正则**，宁可宽一点——**宁可假阳性让 agent 去判断，也不要假阴性把真漂移漏掉**。

## 参考

- 约定登记表：`~/Documents/projects/project-scaffold/conventions.toml`
- 检测器：`~/Documents/projects/project-scaffold/scaffold-doctor`（`--json` 可机读）
- 设计说明：`~/Documents/projects/project-scaffold/README.md`
