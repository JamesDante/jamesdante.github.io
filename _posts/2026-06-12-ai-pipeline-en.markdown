---
layout: post
title: "Using AI to Write AI Tutorials: Generating 21 Bilingual Chapters with One Command"
date: 2026-06-12 10:00:00 +0800
categories: AI Engineering
lang: en
ref: ai-pipeline
---

I recently built something with a recursive feel to it: a multi-agent pipeline that generates a course on AI-assisted coding.

Feed it a curriculum outline, and it automatically produces 21 chapters of bilingual technical documentation — complete with code examples, architecture diagrams, review, and auto-fixing. It's open source: [github.com/jamesdante/ai-coding-101](https://github.com/jamesdante/ai-coding-101)

## Pipeline Structure

Seven agents, running in sequence, each with a single responsibility:

```
Curriculum Agent   ← parses the outline, plans chapter structure
    ↓
Spec Agent         ← generates a JSON spec for each chapter
    ↓
Writer Agent       ← outputs English MDX
    ↓
Translator Agent   ← translates to Chinese
    ↓
Reviewer Agent     ← scores on 8 dimensions (100 points total)
    ↓
Fixer Agent        ← automatically fixes low-scoring issues
    ↓
Re-Reviewer        ← verifies the fixes
```

One command runs the whole thing:

```bash
python generate_site.py --outline outline.md
```

## The Reviewer's 8 Scoring Dimensions

The Reviewer Agent scores each chapter across eight dimensions, out of 100:

| Dimension | What it checks |
|-----------|----------------|
| Spec compliance | Does the content match the JSON spec from Spec Agent? |
| Technical accuracy | Is the code correct and runnable? |
| Clarity | Is the writing readable and well-structured? |
| MDX validity | Is the component syntax valid? |
| Completeness | Does the chapter fully cover what the outline requires? |
| Conceptual depth | Are the underlying principles sufficiently explained? |
| End-to-end examples | Is there a complete, runnable example? |
| Failure patterns | Are typical errors and debugging approaches covered? |

Each dimension has explicit deduction rules. Uninitialized variables in code, missing architecture diagrams, absent failure-pattern tables — all get flagged and handed to the Fixer Agent.

## Bugs Encountered Along the Way

A few concrete problems that came up, all now fixed:

**Writer inventing next-chapter titles**: The Writer Agent would sometimes make up the names of adjacent chapters. The fix was to inject a `next_chapter_title` field into each spec so it has a ground truth to reference.

**Chinese translation producing Korean characters / garbled text**: Added a hard constraint to the Translator prompt requiring output in Simplified Chinese only, with English preserved for proper nouns.

**Reviewer output getting truncated**: The scoring JSON is fairly complex. Bumping `max_tokens` from 4096 to 6144 solved it.

**Unreachable methods in generated code**: Added a dedicated technical-accuracy deduction rule to the Reviewer to catch this class of bug.

## Course Content

21 chapters, covering:

Prompt engineering → Full-stack AI-assisted development → Tool Calling / MCP / Multi-agent systems / Coding agents → Real-world projects (SaaS / Chat App / Agent Platform) → RAG / Large-context handling / Enterprise AI

All chapters bilingual, Chinese and English in sync.

## Tech Stack

- **Site**: Docusaurus v3 with i18n for language switching
- **Pipeline**: Python, ~1,300 lines
- **LLM**: Anthropic API (swappable for Ollama to run locally)
- Supports resuming from a checkpoint, running specific chapters only, `--skip-review`, and other flags

## The Project Is a Demonstration of What It Teaches

A multi-agent system solving a real problem. Each agent has a single responsibility, hands off structured JSON to the next, and the whole pipeline has review, repair, and verification built in. The design mirrors exactly the principles the course teaches.

Code and prompts are fully open: [github.com/jamesdante/ai-coding-101](https://github.com/jamesdante/ai-coding-101)
