# project-scaffold

个人项目的**文档脚手架**：让每个新项目一开始就具备「仓库自描述」能力——
状态、决策、给 AI 的上下文都写在文件里，不依赖对话记忆。

配套的规范做法来自 `exam-sim` / `webpokerdealer` 等项目的沉淀。

## 用法

```bash
cd ~/Documents/projects
./new-project <项目名> "一句话简介" ["技术栈要点"]

# 例：
./new-project hello-finance "个人记账小工具" "Python 3.12 + FastAPI + SQLite"
```

会从 `_template/` 生成一个项目目录并 `git init` + 首次提交。

> `~/Documents/projects/new-project` 是指向本脚本的符号链接；
> 脚本用真实路径定位 `_template/`，因此从任何目录调用都行，
> 但**新项目建在当前工作目录下**。

## 生成的骨架

| 文件 | 回答的问题 |
|---|---|
| `README.md` | 这是什么、怎么跑起来（对外） |
| `AGENTS.md` | 技术栈、命令、代码约定、提交规范、红线（给 AI / 接手者） |
| `docs/ROADMAP.md` | **唯一状态真相**：里程碑、当前状态、下一步、已知局限 |
| `docs/DECISIONS.md` | 关键架构决策记录（ADR-lite，参考 [MADR](https://adr.github.io/madr/)） |
| `.gitignore` | Python 通用 + `.env` + 编辑器 |

占位符：`{{PROJECT_NAME}}`、`{{PROJECT_DESC}}`、`{{TECH_STACK}}`。

## 约定漂移：一个问题，两半解法

**问题**：模板更新后，已有项目不会自动跟进。根因是——
**git 的版本单位是「仓库」，而约定文件属于「一族仓库」**，
git 没有原语表达「这个文件是模板 X 在版本 Y 的实例」。

**在 LLM 时代，这个陈旧问题变成了正确性问题**：agent 会把过期的约定文件
当**真理**执行，而且不报错。这正是「恢复旧会话会用错旧决策」的同一个机制。

### 分工：确定性的归工具，语义的归 agent

```
scaffold-doctor  ──检测──▶  哪条约定在哪个项目缺失（确定性）
                          │
                          ▼
    agent（/skill:sync-conventions）──判断──▶ 同步 / 适配 / 跳过（语义）
```

**为什么不做一个自动同步工具？** 因为同步本质上需要语义判断：

- 同一条约定在不同项目里**措辞本就不同**（正文项目写「没做完」，笔记项目写「没写完」）；
- 有些漂移是**误报**（内容在别的标题下）；
- 有些约定是**内容性**的，需要人提供素材，agent 编不出来。

所以工具只提供**事实**，判断交给 agent。这条经验也来自实测：检测器第一版就误报了
`webpokerdealer` 的「下一步」小节。

### 检测器

```bash
./scaffold-doctor            # 人类可读
./scaffold-doctor --json     # 给 agent / 脚本消费
```

读取 `conventions.toml`，扫描 `~/Documents/projects` 下含 `AGENTS.md` 的项目。
退出码 `0` = 无漂移，`1` = 有漂移。

### 约定的两种类型

| kind | 含义 | agent 能做吗 |
|---|---|---|
| `structural` | 补标题 / 补一条规则 | ✅ 能独立完成 |
| `content` | 需要真实素材（如「考虑过的其他选项」） | ⚠️ 只能搭骨架，内容待补 |

### 新增一条约定

两处改动，缺一不可：

1. 在 `_template/` 的对应文件里**写清楚**（这才是真相源）
2. 在 `conventions.toml` 登记一条 `pattern`

`pattern` 用**正则**，宁可宽一点：**宁可假阳性让 agent 去判断，也不要假阴性把真漂移漏掉。**

### 安装技能

```bash
./install-skill     # 复制 skills/ 到 ~/.pi/agent/skills/
```

然后在 pi 里用 `/skill:sync-conventions`。

> 用复制而非符号链接，避免不同实现对符号链接的处理差异；改完技能要重新跑一次。

## 目录结构

```
project-scaffold/
├── new-project              # 生成新项目
├── _template/               # 骨架真相源
├── scaffold-doctor          # 约定漂移检测（确定性）
├── conventions.toml         # 约定登记表
├── install-skill            # 安装技能到 pi
└── skills/
    └── sync-conventions/
        └── SKILL.md         # 同步技能（语义判断）
```

## 维护

- 改规范 → 改 `_template/`，新项目即生效。
- 已有项目 → 跑 `./scaffold-doctor`，再用 `/skill:sync-conventions` 处理漂移。
