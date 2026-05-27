# When Is SDD the Right Tool? — An Analysis of github/spec-kit

> Based on analysis of [github/spec-kit](https://github.com/github/spec-kit) and its documentation.

---

## What SDD Actually Is

The core thesis is a **"power inversion"**: for decades, specifications served code — they were scaffolding built and then discarded once coding began. SDD inverts this: code serves specifications, with the PRD as the source that generates implementation rather than guides it.

This is a real and significant shift. The spec becomes a *living, versioned artifact*. Code is regenerated output, not the ground truth.

The workflow is structured as:

```
idea → iterative dialogue with AI → comprehensive PRD → implementation plan → task breakdown → code generation
```

The spec-kit CLI formalizes this with a command chain:

```
/speckit.constitution → /speckit.specify → /speckit.clarify → /speckit.plan → /speckit.tasks → /speckit.implement
```

---

## Where SDD Genuinely Excels

### 1. Greenfield Projects with Articulable Requirements

SDD targets "0-to-1 development" (greenfield) as its primary use case — starting with high-level requirements, generating specifications, planning implementation steps, and building production-ready applications. When you know *what* you want to build but not *how*, this is the strong case.

### 2. High Requirement Volatility

SDD transforms requirement changes from obstacles into normal workflow. Change a core requirement in the PRD, and affected implementation plans update automatically. Modify a user story, and corresponding API endpoints regenerate. This isn't just about initial development — it's about maintaining engineering velocity through inevitable changes.

### 3. Team Processes Where Specs Are the Review Artifact

The workflow supports a team process where team-reviewed specifications are expressed and versioned, created in branches, and merged. This maps naturally onto PR-based workflows.

### 4. Enterprise Constraints

Organizational constraints — cloud providers, tech stacks, engineering practices, enterprise design systems, and compliance requirements — integrate into every specification via the `constitution.md` file, which encodes these permanently.

### 5. Iterative Brownfield Enhancement

SDD explicitly supports "iterative enhancement / brownfield modernization" — adding features iteratively, modernizing legacy systems, adapting processes.

### 6. Parallel Exploration

You can generate multiple implementation approaches from the same specification to explore different optimization targets: performance, maintainability, user experience, cost.

---

## Where SDD Is the Wrong Tool

> This is where the repo's own documentation undersells its limitations. The documentation is almost entirely positive and buries the real constraints.

### 1. Exploratory / Research-Driven Work

SDD assumes you can articulate *what* you want before you build. If you're doing research, a spike, or genuinely don't know the requirements until you've tried something, SDD's overhead is pure waste. You end up writing speculative specs and then discarding them — the inversion doesn't work when the spec itself is unknown.

### 2. Well-Understood, Routine Patterns

A CRUD endpoint, a CLI script, a database migration — these don't benefit from a `constitution` + `specify` + `plan` + `tasks` chain. Direct prompting is faster and produces identical results. The repo implicitly admits this by noting the workflow is designed for complex features, not boilerplate.

### 3. Pure Refactoring

If behavior is unchanged and you're restructuring internals for clarity, performance, or maintainability, there is no spec delta to drive. SDD has nothing to say here.

### 4. Safety-Critical and Adversarial Contexts

SDD explicitly relies on "heavy reliance on advanced AI model capabilities for specification interpretation" — that's a direct quote from the core philosophy, and also its red flag. In safety-critical, financial, medical, or adversarially-constrained environments, AI-generated implementation plans require formal verification, not trust. The workflow produces artifacts that *look* complete; whether they are correct is a separate question that SDD's tooling does not address.

### 5. Zero-Hallucination Tolerance

The spec-kit docs and detailed walkthrough repeatedly warn that the agent can be "over-eager," add unrequested components, and research the wrong thing. The docs explicitly state: *"Claude Code might be over-eager and add components that you did not ask for. Ask it to clarify the rationale."*

The entire process requires an active human auditor at every phase. If your team cannot catch AI hallucinations in generated plans and code, SDD's multi-step pipeline makes the problem *worse*, not better — errors propagate from spec through plan through tasks through implementation.

---

## Structural Weaknesses the Project Doesn't Advertise

### The "Executable Spec" Claim Is Overstated

The docs acknowledge that "practicing SDD requires assembling existing tools and maintaining discipline throughout the process." There is no runtime enforcement that the spec stays in sync with code. The spec is a Markdown file. Nothing prevents code drift, and the claim that "maintaining software means evolving specifications" is an aspiration, not a mechanism.

### The Process Is Token-Expensive

A single feature goes through `constitution → specify → clarify → plan → research → tasks → implement`, each generating multi-KB Markdown artifacts and multiple agent interactions. For a team, this is a real cost and latency that needs justification before adoption.

### The Methodology Is GitHub Copilot-Centric

The default agent integration is GitHub Copilot. Claude Code is listed as a supported integration, but the detailed walkthrough examples and tooling defaults are written for Copilot. Claude Code integration works, but you are using a framework designed and optimized for a different agent.

---

## Decision Reference

| Situation | Use SDD? | Reason |
|---|---|---|
| Greenfield product / 0→1 | ✅ Yes | Core use case |
| Complex, articulable requirements | ✅ Yes | Spec→plan→code chain pays off |
| Iterative brownfield features | ✅ Yes | Spec evolves; implementation regenerates |
| Team with review gates | ✅ Yes | Versioned specs map to PRs naturally |
| Enterprise / org constraints | ✅ Yes | `constitution.md` encodes them once |
| High requirement volatility | ✅ Yes | Pivot = spec update, not manual rewrite |
| Parallel implementation exploration | ✅ Yes | Same spec, multiple implementations |
| Exploratory spike / throwaway POC | ❌ No | You learn the what by building first |
| Well-understood, boilerplate task | ❌ No | Overhead exceeds benefit |
| Single-shot disposable script | ❌ No | No iteration planned |
| Safety-critical / high-assurance | ❌ No | AI output needs formal verification |
| Pure internal refactor | ❌ No | No spec delta to drive |
| Research / emergent domain | ❌ No | No stable spec possible before coding |
| Zero-hallucination tolerance | ⚠️ Caution | Active auditing required at every step |

---

## The Honest Summary

SDD is a genuine improvement over pure vibe coding for **complex, planned features where requirements can be articulated in advance**. It adds structural discipline that catches ambiguities early and creates a reviewable, versioned artifact trail.

It is overkill — sometimes counterproductive — for exploratory work, routine tasks, refactoring, and any context where AI output accuracy cannot be tolerated without independent verification.

The framework's strength is also its constraint: **it only works when you can describe what you want before you build it.**

---

### Workshop Note (Java 25 / Gradle 9 / Number Guessing Game)

The Number Guessing Game is intentionally simple, which means SDD's value in the workshop is *pedagogical*, not practical. The game itself doesn't warrant the full pipeline — that's fine and appropriate for teaching purposes, but worth being explicit about with participants so they don't over-apply the methodology to small tasks.
