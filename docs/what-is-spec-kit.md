# What is Spec Kit?

> A guide to Spec-Driven Development and the `github/spec-kit` toolkit.  
> Source: [github.com/github/spec-kit](https://github.com/github/spec-kit)

---

## 1. What Is Spec Kit?

Spec Kit is an open-source toolkit published by GitHub that enables **Spec-Driven Development (SDD)** — a methodology that inverts the traditional relationship between specifications and code.

| Traditional development | SDD |
|---|---|
| Specifications *serve* code — they guide implementation, then are discarded | Code *serves* specifications — the spec is the source of truth; code is its regenerated output |

In practice, spec-kit provides a CLI tool (`specify`) and a set of slash commands that work inside AI coding agents (Copilot, Claude Code, Gemini CLI, and 30+ others). These commands walk a team through a structured pipeline from idea to running implementation.

---

## 2. The SDD Pipeline

Every feature in spec-kit moves through the same ordered stages:

| Step | Command | What it produces |
|---|---|---|
| 1 | `/speckit.constitution` | `constitution.md` — project principles, tech standards, quality gates |
| 2 | `/speckit.specify` | `spec.md` — user stories, functional requirements, acceptance criteria |
| 3 | `/speckit.clarify` | Clarifications section — fills gaps before planning begins |
| 4 | `/speckit.plan` | `plan.md`, `data-model.md`, `api-spec.json`, `research.md` |
| 5 | `/speckit.tasks` | `tasks.md` — ordered, dependency-aware, parallelism-marked task list |
| 6 | `/speckit.implement` | Working implementation built from the task list |

---

## 3. Core Philosophy

SDD rests on five principles:

- **Specifications as the lingua franca** — the spec is the primary artifact; code is its expression in a particular language and framework.
- **Executable specifications** — specs must be precise, complete, and unambiguous enough to generate working systems.
- **Continuous refinement** — consistency validation happens as an ongoing process, not a one-time gate.
- **Research-driven context** — research agents gather technical context throughout, investigating library compatibility and organizational constraints.
- **Bidirectional feedback** — production metrics and incidents become inputs for specification evolution.

---

## 4. When to Use SDD

SDD is most valuable when requirements can be articulated before building begins. It pays its overhead back in reduced rework, better team alignment, and a versioned audit trail.

### Strong fits

- **Greenfield / 0→1 projects** — no existing codebase to constrain decisions.
- **Complex features with articulable requirements** — you know the *what*; the pipeline handles the *how*.
- **Iterative brownfield enhancement** — spec evolves; implementation regenerates.
- **Team processes with review gates** — specs are versioned, branched, and merged like code.
- **Enterprise or compliance-constrained development** — tech stack and policies baked into `constitution.md`.
- **High requirement volatility** — pivoting means updating the spec, not manually rewriting code.
- **Parallel implementation exploration** — same spec, multiple candidate implementations.

### Poor fits

- **Exploratory spikes or throwaway POCs** — you learn the requirements by building; no stable spec is possible.
- **Boilerplate / well-understood patterns** — a CRUD endpoint does not warrant a full pipeline.
- **Single-shot disposable scripts** — no iteration planned; overhead is not justified.
- **Pure internal refactoring** — behaviour is unchanged; there is no spec delta to drive anything.
- **Research or emergent domains** — the *what* is unknown until coding reveals it.
- **Safety-critical or adversarial contexts** — AI-generated plans require formal verification, not trust.
- **Zero-hallucination tolerance** — every phase requires active human auditing; the pipeline amplifies errors if that auditing is absent.

---

## 5. Decision Reference

| Situation | Use SDD? | Reason |
|---|---|---|
| Greenfield product / 0→1 | ✅ Yes | Core use case for the methodology |
| Complex, articulable requirements | ✅ Yes | Spec→plan→code chain pays off |
| Iterative brownfield features | ✅ Yes | Spec evolves; implementation regenerates |
| Team with review gates | ✅ Yes | Versioned specs map to PRs naturally |
| Enterprise / org constraints | ✅ Yes | `constitution.md` encodes them once |
| High requirement volatility | ✅ Yes | Pivot = spec update, not manual rewrite |
| Exploratory spike / throwaway POC | ❌ No | You learn the *what* by building first |
| Well-understood boilerplate task | ❌ No | Overhead exceeds benefit |
| Single-shot disposable script | ❌ No | No iteration planned |
| Safety-critical / high-assurance | ❌ No | AI output needs formal verification |
| Pure internal refactor | ❌ No | No spec delta to drive |
| Zero-hallucination tolerance | ⚠️ Caution | Active auditing required at every step |

---

## 6. Known Limitations

### The "executable spec" claim is overstated

Spec-kit acknowledges that the methodology requires "assembling existing tools and maintaining discipline." There is no runtime mechanism that keeps code in sync with the spec. The spec is a Markdown file. Code drift is possible and likely without team discipline.

### The process is token-expensive

A single feature traverses six phases, each producing multi-kilobyte Markdown artifacts and multiple agent interactions. For large teams, this represents real cost and latency that must be justified by the value of the resulting artifact trail.

### AI hallucinations propagate downstream

Errors introduced in the plan phase are carried into the tasks phase and then into the implementation. The spec-kit documentation explicitly warns that the agent can be "over-eager" and add unrequested components. Human review at each phase is not optional — it is structural.

### Copilot-centric design

The detailed walkthrough examples, default integrations, and tooling defaults are written for GitHub Copilot. Claude Code is supported but is being used within a framework optimised for a different agent.

---

## 7. Summary

> SDD is a genuine improvement over pure vibe coding for **complex, planned features where requirements can be articulated in advance**. It adds structural discipline, catches ambiguities early, and creates a reviewable, versioned artifact trail.
>
> The framework's strength is also its constraint: **it only works when you can describe what you want before you build it.**
