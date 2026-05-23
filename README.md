# skill-readable-output

**Distill long AI conversations into structured, reviewable HTML pages.**

> Turn hours of discussion into a 5-minute read that rebuilds the thinking context — not a transcript, but a decision record.

[中文文档](README_zh.md)

---

## What It Does

When you have a long conversation with an AI assistant — debugging a system, designing architecture, exploring research directions, or making product decisions — the insights get buried in chat history. Three months later, you'll ask the same questions again.

This skill extracts the **decisions**, **reasoning**, and **action items** from your conversation and renders them as a clean, self-contained HTML page you can revisit anytime.

## Quick Start

### For Claude Code / Kiro

1. Copy `SKILL-readable-output.md` into your project's `.claude/skills/` directory:

```bash
mkdir -p .claude/skills
cp SKILL-readable-output.md .claude/skills/
```

2. After a productive conversation, just say:

```
整理一下这次讨论
```

or

```
/readable
```

3. The skill will ask 2-3 quick questions about context, then generate a structured HTML file and open it in your browser.

### For Other AI Assistants

The skill file is a structured prompt. You can adapt it for any AI assistant that supports system prompts or custom instructions. The core logic (phases, structure templates, anti-patterns) is transferable.

## Features

### 6 Scenario Templates

| Scenario | Structure | Best For |
|----------|-----------|----------|
| Research Design | Timeline (Problem → Options → Selection) | Academic discussions, study design |
| Architecture Decision (ADR) | Context → Decision → Consequences | System design, tech choices |
| Debug Postmortem | Symptom → Investigation → Root Cause → Fix | Troubleshooting sessions |
| Product Direction | Problem → Solutions → Tradeoffs → MVP | Product/feature discussions |
| Tech Comparison | Weighted matrix (Dimensions × Candidates) | Library/framework selection |
| Action Extraction | Priority-sorted TODO list | Quick wrap-ups |

### 4 Visual Styles

| Style | Look | Default For |
|-------|------|-------------|
| Research Narrative | Warm white + timeline + callouts | Academic, research |
| Technical Document | GitHub-style + code blocks + badges | Code, architecture |
| Product Memo | Cards + tags + progress indicators | Product, features |
| Decision Brief | Minimal black/white + priority labels | Quick decisions |

### Smart Defaults

- Say "整理一下" with no further input → auto-applies "self-review + decision record + standard depth"
- Style auto-matches scenario (ADR → Technical Document, etc.)
- Volume guard: warns you if content is too thin for "full version"

### Built-in Quality Gates

- **3-point cap**: No more than 3 core insights per document (forces prioritization)
- **Half-cut test**: "If you could only keep half, which stay?" — survivors are the real skeleton
- **4-question self-check**: Readable without context? Decisions + reasons present? Not a transcript? TODOs actionable?
- **Anti-pattern list**: 11 documented failure modes to avoid

## Output Structure

Every generated HTML includes:

```
┌─────────────────────────────────┐
│ Config Bar (scenario/depth/style)│
├─────────────────────────────────┤
│ TL;DR (2-3 sentences)           │
├─────────────────────────────────┤
│ Main Content                    │
│ (structure varies by scenario)  │
├─────────────────────────────────┤
│ Action Items (prioritized)      │
├─────────────────────────────────┤
│ References (links/papers/docs)  │
├─────────────────────────────────┤
│ Open Questions + Boundaries     │
└─────────────────────────────────┘
```

## Customization

### Adding Your Own Style

Add a new CSS `:root` block in the skill file under "CSS 设计 token" section. The HTML skeleton is shared across all styles — only variables change.

### Adjusting File Paths

Edit the "文件路径（智能路由）" section to match your project structure. Default:

```
Academic    → docs/discussions/discussion_{topic}_{date}.html
Technical   → {project}/docs/adr_{topic}_{date}.html
Product     → {project}/docs/decision_{topic}_{date}.html
```

### Extending Scenarios

Add rows to the "阶段 4 · 选主结构" table. Each scenario needs: a name, a recommended structure pattern, and optionally a default style mapping.

## How It Works

```
User triggers → Phase 1: Scenario confirmation (2-3 questions)
             → Phase 1.5: Volume self-check (anti-bloat guard)
             → Phase 2: Define endpoint (what reader gets)
             → Phase 3: Extract ≤3 core points (half-cut test)
             → Phase 4: Select structure template
             → Phase 5: Write (TL;DR → content → TODOs → refs → exit)
             → Phase 6: Self-check 4 questions
             → Output HTML + auto-open in browser
```

## When NOT to Use

- Conversation < 5 turns (just write a one-liner note)
- Pure code implementation with no decision discussion
- Content already structured elsewhere (Jira, Notion, Linear)
- You want markdown, not HTML (just say so)
- Discussion is still ongoing (wait until it concludes)

## Contributing

Issues and PRs welcome. If you've adapted this for a different AI assistant or added a new scenario template, I'd love to see it.

## License

[MIT](LICENSE)
