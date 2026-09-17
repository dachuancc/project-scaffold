# project-scaffold

个人项目的**文档脚手架**：让每个新项目一开始就具备「仓库自描述」能力——
状态、决策、给 AI 的上下文都写在文件里，不依赖对话记忆。

配套的规范做法来自 `exam-sim` / `webpokerdealer` 两个项目沉淀出的范式。

## 用法

```bash
cd ~/Documents/projects
./new-project <项目名> "一句话简介" ["技术栈要点"]

# 例：
./new-project hello-finance "个人记账小工具" "Python 3.12 + FastAPI + SQLite"
```

会从 `_template/` 生成一个项目目录并 `git init` + 首次提交。

> `~/Documents/projects/new-project` 是指向本脚本的符号链接，方便直接 `./new-project` 调用；
> 脚本用真实路径定位 `_template/`，因此从任何目录调用都行，但**新项目建在当前工作目录下**。

## 生成的骨架

| 文件 | 回答的问题 |
|---|---|
| `README.md` | 这是什么、怎么跑起来（对外） |
| `AGENTS.md` | 技术栈、命令、代码约定、提交规范、红线（给 AI / 接手者） |
| `docs/ROADMAP.md` | **唯一状态真相**：里程碑、当前状态、下一步、已知局限 |
| `docs/DECISIONS.md` | 关键架构决策记录（ADR-lite），编号、只增不改 |
| `.gitignore` | Python 通用 + `.env` + 编辑器 |

占位符：`{{PROJECT_NAME}}`、`{{PROJECT_DESC}}`、`{{TECH_STACK}}`。

## 维护

- 改规范 → 直接改 `_template/` 里的文件，新项目即生效。
- 已有项目**不会**自动同步，需手动对齐（例如补「提交信息规范」一节）。
