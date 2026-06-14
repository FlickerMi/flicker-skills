# Project Mapper（项目地图生成器）

**[English](./README.md) | [中文](./README.zh-CN.md)**

一个 Claude skill，扫描代码库并生成两个精简的**按需加载**参考文档，让 AI 编码 agent 一次读取即可定位方向，而不必在每个任务时全量扫描整个项目。

## 为什么

`CLAUDE.md` 每个会话开头都会自动加载进 context —— 写在里面的每一行都是"每次都付费"的固定成本。本 skill 产出的两个文档是**按需加载**（agent 仅在需要时才读），空闲时零成本。

| 文档 | 回答什么 | 用途 |
|------|----------|------|
| `docs/code-map.md` | "X 在哪 / 如何串联" | **定方向** —— 在 agent 搜索前先把它指向正确的子目录 |
| `docs/debug-playbook.md` | "这个症状 → 看这里 → 查这个" | **排查** —— 仅在 debug 时才加载 |

分工：**code-map 定方向（粗定位），实时 grep/LSP 找精确行。** 文档只停在模块/架构级 —— 绝不写到函数级，函数级会随代码漂移（drift）并误导 agent。

## 它做什么

1. **高效扫描** —— 读 manifest（`package.json`、`pom.xml` 等）、目录树、关键入口文件，用 grep/glob 枚举 route/controller/service，而不是逐个读文件；对于陌生项目，还会从 `git log --oneline -100`、`CHANGELOG.md`、`.github/ISSUE_TEMPLATE/` 拉取历史信号。
2. **生成 `docs/code-map.md`**（目标 150–250 行）—— 目录映射（标注 `active` / `legacy`）、入口启动流程、核心调用链、关键模块、外部集成、约定，以及最重要的 **"症状 → 区域"速查表**（目标 15–25 行，业务面 × 错误面双轴混合）。
3. **生成 `docs/debug-playbook.md`**（目标 100–200 行）—— 运行/构建/测试命令、日志位置、**有实际证据支撑**的常见故障场景（按"启动期 / 请求期 / 后台任务"分组）、诊断工具箱、项目特有坑。
4. **在 `CLAUDE.md` 写入触发契约**（纯文本，非 `@import`）—— 告诉未来的 agent **什么时候必须读这两份文档**，否则 agent 默认还是会先全项目 grep。

## Monorepo

项目根目录若包含多个子项目（`frontend/` + `backend/`、`apps/*` + `packages/*` 等），会在 **git 仓库根目录**生成**一份**覆盖全部子项目的文档，绝不写进任何子项目目录 —— 即使子项目自带 `docs/` 也不会（那会让在根目录工作的 agent 找不到）。

## 安装

在 Claude（Claude Code / Claude.ai 的 skills）中安装 `project-mapper.skill` 文件。安装后会自动触发。

## 推荐用法

### 1. 生成文档

在任意项目根目录，跟 Claude Code 说：

- "生成 code map 和 debug playbook"
- "Generate a code map and debug playbook in English"
- "帮 AI 理解这个仓库的结构"

skill 会自动跑完扫描 → 生成文档 → 给出 CLAUDE.md 接入建议。仓库中英混用且未指定时，会先问一次用哪种语言。

### 2. 在 `/debug` 命令里固化触发（最强推荐）

光生成文档不够 —— agent 不一定会主动去读。**最稳的做法是把"读 playbook"硬编码进 `/debug` 命令本身**：命令一触发就读，不依赖 agent 自觉，是触发时机最早的方式。

新建 `.claude/commands/debug.md`（项目级）或 `~/.claude/commands/debug.md`（全局）：

```markdown
---
description: 帮我定位项目里的 bug
---

按下面这个顺序处理用户报告的 bug，不要跳步：

1. **第一步，读 `docs/debug-playbook.md`** —— 把用户描述的症状对照 playbook 的 "Common scenarios"，看是否命中已知场景。
2. 若命中：按 playbook 列的诊断步骤走；定位到目录后，跳 `docs/code-map.md` 拿精确路径。
3. 若没命中：去 `docs/code-map.md` 的 "Where to look" 表查最接近的业务面或错误面，定位到子目录后再开始 grep。**禁止先全项目 grep。**
4. 锁定嫌疑文件后，用 Read 读具体代码，提出修复方案。

用户报告的 bug：$ARGUMENTS
```

之后排查 bug 直接 `/debug "登录后立刻 401"`，agent 第一步就被强制带去读 playbook。

### 3. 在 CLAUDE.md / AGENTS.md 写入触发契约

适用于"用户没用 `/debug`、直接描述 bug"的场景。在 `CLAUDE.md` 或 `AGENTS.md` 加这一段（**不要用 `@import`**，否则会被全量自动加载，等于白做）：

```markdown
## Reference docs (read on demand)

按需加载 —— 只在触发条件命中时，用 Read 工具读取。

- **`docs/code-map.md`** —— 架构 / 事物所在位置。
  **探索任何不熟悉的区域前，先读这份。** 不要直接全项目 grep —— 先用 "Where to look" 表定位到子目录，只在该子目录里 grep。

- **`docs/debug-playbook.md`** —— 症状 → 区域 → 诊断步骤。
  **任何 bug、测试失败、异常行为的排查，第一步先读这份。** 在形成假设或开始大范围搜索之前，先把用户的症状对照 playbook 里的场景。
```

### 4. 何时重新生成

当**架构**变化时再跑一次，不必每次 commit 都更新：

- 新增 / 拆分 / 合并模块
- 新接入第三方集成或数据库
- 大规模重构、目录结构调整
- 单仓拆 monorepo，或反之

模块级文档本身就稳定，这正是它的价值。日常开发（改 bug、加字段、小功能）不会让文档失准。

## 语言

输出语言为**中文或英文**，按以下优先级决定：

1. **你显式指定的** —— 例如 "用中文生成" / "output in English"，最高优先级。
2. **仓库现有文档** —— 未指定时，匹配 `CLAUDE.md` / `AGENTS.md` / `README` 的语言。
3. **回退** —— 当前对话所用的语言。

## 结构

```
project-mapper/
├── SKILL.md                          # 工作流（供 agent 读取）
├── README.md                         # 英文说明
├── README.zh-CN.md                   # 本文件（中文）
└── references/
    ├── code-map-template.md          # docs/code-map.md 的结构模板
    └── debug-playbook-template.md    # docs/debug-playbook.md 的结构模板
```
