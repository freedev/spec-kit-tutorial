# Spec-Driven Development: Workshop Toolkit Guide

> Conversation summary — April 25, 2026

---

## Context

The goal is to create a **spec-driven workshop** that explains how to start a project from scratch using the most valuable and relevant SDD toolkits available today.

---

## What Is Spec-Driven Development?

Spec-driven development (SDD) is a methodology where formal, detailed specifications serve as the **source of truth** that guides AI agents to generate, test, and validate code. Instead of coding first and writing docs later, you start with a spec — a structured, behavior-oriented artifact written in natural language — and let that spec drive implementation, checklists, and task breakdowns.

The core workflow follows four phases:

```
Specify → Plan → Tasks → Implement
```

> **Important caveat:** SDD shines when starting a new project from scratch, but as the application grows, specs can drift and slow development. For large existing codebases, SDD is mostly unusable. A good workshop should be explicit about *when* SDD adds value and when it doesn't.

---

## The Toolkit Landscape (2026)

The leading spec-driven tools separate into two categories:

- **Living-spec platforms** — keep documentation synchronized with code as agents work
- **Static-spec tools** — structure requirements upfront but require manual reconciliation when implementation diverges

---

## Toolkits Ranked: Most to Least Recommended

### 1. 🥇 GitHub Spec Kit
- **Type:** Static (markdown), open-source (MIT)
- **Agent support:** 8+ agents (Copilot, Claude Code, Gemini CLI, and more)
- **Best for:** Open-source cross-agent standardization, learning the canonical SDD workflow
- **Price:** Free

The **reference standard** for spec-driven development. Most documented, most community resources, most extensions. Provides a structured 4-phase workflow with a CLI (`specify`), templates, and a "constitution" for immutable project principles. The unavoidable foundation for any SDD workshop.

---

### 2. 🥈 Amazon Kiro
- **Type:** Static (EARS notation), agentic IDE
- **Agent support:** Claude models (AWS-native)
- **Best for:** Formal requirements, AWS-native greenfield projects
- **Price:** Free (50 credits/month)

Uses **EARS (Easy Approach to Requirements Syntax)** — a formal, industry-tested notation for unambiguous requirements. Guides users through three stages: requirements, design, and task creation. Excellent for demonstrating structured, rigorous requirements engineering beyond just AI tooling.

---

### 3. 🥉 BMAD-METHOD
- **Type:** Static (docs-as-code), open-source
- **Agent support:** 12+ role-based agents, IDE-agnostic
- **Best for:** Team/role-based workflows, enterprise planning
- **Price:** Free

Assigns distinct AI agents to roles (PM, architect, developer, QA), mirroring how real software teams are structured. Framework-heavy but highly instructive for teaching collaborative, multi-agent thinking.

---

### 4. BDD / Gherkin (Cucumber, Behave)
- **Type:** Executable specifications
- **Best for:** Foundation layer, stakeholder collaboration, automated testing
- **Price:** Free / open-source

The **historical foundation** all modern SDD builds on. Gherkin's `Given/When/Then` format produces scenarios that serve dual purposes: documentation stakeholders can read *and* automated tests that verify code. Teaching this first grounds participants in decades of proven practice.

---

### 5. OpenSpec
- **Type:** Semi-living (delta markers), open-source
- **Agent support:** 20+ agents (most agent-agnostic option)
- **Best for:** Brownfield/iterative changes on existing codebases
- **Price:** Free

The most practical tool for the real world — most projects aren't greenfield. OpenSpec's delta markers track changes iteratively, making it the right choice when the codebase already exists and teams need to evolve it incrementally.

---

### 6. Augment Intent
- **Type:** Living (bidirectional), standalone desktop workspace
- **Agent support:** BYOA (Claude Code, Codex, OpenCode + native Auggie agent)
- **Best for:** Multi-service complex codebases, teams needing living-spec synchronization
- **Price:** $60/month (Standard, up to 20 users)

Technically the most advanced tool. Specs update **continuously** as agents implement changes — no manual reconciliation needed. A Coordinator Agent fans work out to specialist agents (Investigate, Implement, Verify, Critique, Debug, Code Review), all sharing the same living spec. Best used as a "where this is heading" demo rather than a hands-on exercise in a workshop.

---

### 7. Tessl *(Experimental)*
- **Type:** Spec-as-source (radical approach)
- **Status:** Private beta as of 2025
- **Best for:** Conceptual debate, forward-looking discussion

Takes the most radical position: the specification itself is the maintained artifact, not the code. Code becomes a disposable output. Not yet suitable for hands-on practice, but invaluable for sparking the core philosophical debate: *what is actually the source of truth in a software project?*

---

## Summary Comparison Table

| Rank | Tool | Spec Type | Multi-Agent | Agent Flexibility | Price |
|------|------|-----------|-------------|-------------------|-------|
| 1 | GitHub Spec Kit | Static (markdown) | None (agent-agnostic) | 8+ agents | Free |
| 2 | Amazon Kiro | Static (EARS) | Single + hooks | Claude models | Free (50 credits/mo) |
| 3 | BMAD-METHOD | Static (docs-as-code) | 12+ role-based | IDE-agnostic | Free |
| 4 | BDD / Gherkin | Executable specs | N/A | Framework-native | Free |
| 5 | OpenSpec | Semi-living (delta) | None (agent-agnostic) | 20+ agents | Free |
| 6 | Augment Intent | Living (bidirectional) | Coordinator + specialists | BYOA (4+ agents) | $60/mo |
| 7 | Tessl | Spec-as-source | N/A | N/A | Private beta |

---

## Recommended Workshop Structure

> The most effective teaching order is **not** the same as the ranking. Move participants from foundations → practice → advanced concepts.

| Step | Tool | Focus |
|------|------|-------|
| 1 | BDD / Gherkin | What is a spec? Why does it matter? Spec syntax basics. |
| 2 | GitHub Spec Kit | The canonical modern SDD workflow (4 phases). |
| 3 | Amazon Kiro | Formal, structured requirements with EARS notation. |
| 4 | BMAD-METHOD | Team/role-based agent orchestration. |
| 5 | OpenSpec | The real-world brownfield problem. |
| 6 | Augment Intent + Tessl | The frontier: living specs and the code-vs-spec debate. |

---

## Key Takeaways

- SDD is not about writing exhaustive waterfall documents — it's about giving AI agents unambiguous, structured guidance.
- The spec becomes the **lingua franca** between humans and AI agents.
- Most tools today produce **static specs** that drift from implementation; living-spec tools (Intent) are the emerging solution.
- Match rigor to need: use the minimum specification discipline that removes ambiguity for your context.
- Developers remain essential — their role shifts from manual coding to **steering specifications and reviewing AI outputs**.

---

*Document generated from workshop planning conversation — Claude Sonnet 4.6*
