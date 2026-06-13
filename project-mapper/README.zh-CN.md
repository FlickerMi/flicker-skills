# Project Mapper(项目地图生成器)

**[English](./README.md) | [中文](./README.zh-CN.md)**

一个 Claude skill,扫描代码库并生成两个精简的**按需加载**参考文档,让 AI 编码 agent 一次读取即可定位方向,而不必在每个任务时全量扫描整个项目。

## 为什么

`CLAUDE.md` 每个会话开头都会自动加载进 context —— 写在里面的每一行都是"每次都付费"的固定成本。本 skill 产出的两个文档是**按需加载**(agent 仅在需要时才读),空闲时零成本。

| 文档 | 回答什么 | 用途 |
|------|----------|------|
| `docs/code-map.md` | "X 在哪 / 如何串联" | **定方向** —— 在 agent 搜索前先把它指向正确的子目录 |
| `docs/debug-playbook.md` | "这个症状 → 看这里 → 查这个" | **排查** —— 仅在 debug 时才加载 |

分工:**code-map 定方向(粗定位),实时 grep/LSP 找精确行。** 文档只停在模块/架构级 —— 绝不写到函数级,函数级会随代码漂移(drift)并误导 agent。

## 它做什么

1. **高效扫描** —— 读取 manifest(`package.json`、`pom.xml` 等)和目录树识别技术栈与入口,再用 grep/glob 枚举 route/controller/service,而不是逐个读文件。
2. **生成 `code-map.md`** —— 概览、架构图、目录映射、入口/启动流程、核心调用链、关键模块、外部集成、约定,以及一个"症状 → 区域"的速查索引。
3. **生成 `debug-playbook.md`** —— 运行/构建/测试命令、日志位置、按技术栈定制的常见故障场景、诊断工具箱、排查流程,以及项目特有的坑。
4. **在 `CLAUDE.md` 中加路标** —— 纯文本引用(而非 `@import`),保证文档保持按需加载。

## 语言

输出语言为**中文或英文**,按以下优先级决定:

1. **你显式指定的** —— 例如"用中文生成"/"output in English",最高优先级。
2. **仓库现有文档** —— 未指定时,匹配 `CLAUDE.md`/`AGENTS.md`/`README` 的语言。
3. **回退** —— 当前对话所用的语言。

## 安装

在 Claude(Claude Code / Claude.ai 的 skills)中安装 `project-mapper.skill` 文件。安装后会自动触发。

## 使用

在任意项目里直接说:

- "生成中文版的 code map 和 debug playbook"
- "Generate a code map and debug playbook in English"
- "帮 AI 理解这个仓库的结构"

如果仓库中英混用且你没指定,skill 会先问一次用哪种语言。

## 结构

```
project-mapper/
├── SKILL.md                          # 工作流(供 agent 读取)
├── README.md                         # 英文说明
├── README.zh-CN.md                   # 本文件(中文)
└── references/
    ├── code-map-template.md          # docs/code-map.md 的结构模板
    └── debug-playbook-template.md    # docs/debug-playbook.md 的结构模板
```

## 维护

当**架构**发生变化(新增模块、新增集成、重构)时再重新生成 —— 不必每次提交都更新。模块级文档本身就是稳定的,这正是它的价值。
