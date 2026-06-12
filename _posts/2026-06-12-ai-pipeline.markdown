---
layout: post
title: "用 AI 帮 AI 写教程：一条命令生成 21 章双语技术文档"
date: 2026-06-12 10:00:00 +0800
categories: AI 工程
lang: zh
ref: ai-pipeline
---

最近做了一件有点递归感的事：用多 Agent 流水线来生成一套教 AI 编程的课程。

输入一份大纲，自动输出 21 章双语技术文档，带代码示例、架构图、审校修复全流程。已开源：[github.com/jamesdante/ai-coding-101](https://github.com/jamesdante/ai-coding-101)

直接访问教程：[jamesdante.github.io/ai-coding-101](https://jamesdante.github.io/ai-coding-101/)

## 流水线结构

七个 Agent，顺序执行，职责各自单一：

```
Curriculum Agent   ← 解析大纲，规划章节结构
    ↓
Spec Agent         ← 每章生成 JSON 规范
    ↓
Writer Agent       ← 输出英文 MDX
    ↓
Translator Agent   ← 翻译成中文
    ↓
Reviewer Agent     ← 8 个维度打分（满分 100）
    ↓
Fixer Agent        ← 自动修复低分问题
    ↓
Re-Reviewer        ← 验证修复结果
```

一条命令跑完：

```bash
python generate_site.py --outline outline.md
```

## Reviewer 的 8 个评分维度

Reviewer Agent 对每章从以下维度打分，满分 100：

| 维度 | 说明 |
|------|------|
| 规范遵守 | 是否符合 Spec Agent 输出的 JSON 规范 |
| 技术准确性 | 代码是否正确、可运行 |
| 表达清晰度 | 行文是否易读、结构是否清晰 |
| MDX 合法性 | 组件语法是否合规 |
| 完整性 | 章节内容是否完整覆盖大纲要求 |
| 概念深度 | 是否有足够的原理解释 |
| 端到端示例 | 是否包含完整可运行的示例 |
| 常见失败模式 | 是否覆盖典型错误和调试方式 |

每个维度都有明确的扣分规则。代码里出现未初始化变量、缺少架构图、没有失败模式表——都会被标记出来交给 Fixer Agent 修。

## 踩过的坑

做下来遇到的几个具体问题，都已经修了：

**Writer 乱写"下一章"名称**：Writer Agent 有时会瞎编相邻章节的标题。解法是给每个 Spec 注入 `next_chapter_title` 字段，让它有据可查。

**中文译文出现韩文/乱码**：Translator 的 prompt 加了硬性约束，明确要求只输出简体中文，遇到专有名词保留英文原文。

**Reviewer 输出被截断**：评分 JSON 结构较复杂，`max_tokens` 从 4096 调到 6144 解决。

**代码里混了不可达方法**：Reviewer 新增了技术准确性维度的专项扣分规则来捕捉这类问题。

## 课程内容

21 章，覆盖：

Prompt 工程 → 全栈 AI 辅助开发 → Tool Calling / MCP / 多 Agent / Coding Agent → 实战项目（SaaS / Chat App / Agent Platform）→ RAG / 大规模上下文 / 企业级 AI

全部双语，中英文同步。

## 技术栈

- **网站**：Docusaurus v3 + i18n 中英切换
- **Pipeline**：Python，约 1300 行
- **LLM**：Anthropic API（也支持换成 Ollama 本地跑）
- 支持断点续跑、只跑指定章节、`--skip-review` 等参数

## 这个项目本身就是它教的东西

多 Agent 系统解决一个真实问题。每个 Agent 职责单一，用结构化 JSON 交接，有审校、有修复、有验证。整套流程和课程里讲的设计原则是一致的。

代码和 prompt 全部开源：[github.com/jamesdante/ai-coding-101](https://github.com/jamesdante/ai-coding-101)

教程地址：[jamesdante.github.io/ai-coding-101](https://jamesdante.github.io/ai-coding-101/)
