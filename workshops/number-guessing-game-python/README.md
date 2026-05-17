# SPEC-DRIVEN DEVELOPMENT WORKSHOP

## Building a Number Guessing Game in Python with GitHub Spec Kit

### Python 3.14+ Edition

*A beginner-friendly, hands-on workshop for learning spec-first development from scratch using the real GitHub Spec Kit workflow.*

---

# Introduction

Welcome to this spec-driven development workshop! You will learn how to build software by writing specifications first, then using an AI coding agent to turn those specs into working code. This approach — called Spec-Driven Development (SDD) — helps you think clearly about what your software should do before any code is written.

## What is GitHub Spec Kit?

GitHub Spec Kit (github.com/github/spec-kit) is an open-source toolkit that structures how you work with an AI coding agent. It is NOT a test runner or a code validator. It works by bootstrapping a set of template files and slash commands into your project, which your AI agent then reads and follows to guide you through a structured SDD workflow.

The specify CLI (the command-line tool included in Spec Kit) does one thing: it copies the right template files and slash commands into your project folder for the AI agent you have chosen. After that, all interaction happens through your AI agent.

## The real Spec Kit workflow

| Command | Purpose |
|---|---|
| Step 0 | specify init — bootstrap the project (run once) |
| Step 1 | /speckit-constitution — define project principles |
| Step 2 | /speckit-specify — describe what to build (no tech details yet) |
| Step 3 | /speckit-clarify — optional: refine ambiguities before planning |
| Step 4 | /speckit-plan — choose the tech stack and architecture |
| Step 5 | /speckit-tasks — break the plan into actionable tasks |
| Step 6 | /speckit-implement — have the AI agent execute the tasks |

> **⚠️ Command naming differs by AI agent**
> Claude Code uses hyphen notation:   /speckit-constitution, /speckit-specify, /speckit-plan ...
> GitHub Copilot / Gemini CLI use dot notation:  /speckit.constitution, /speckit.specify ...
> This workshop targets Claude Code. All commands shown use hyphen notation.
> These commands are typed into the agent's chat interface — NOT into the terminal.
> The specify CLI (terminal) is only used once, to initialise the project.

## What you will build

You will build a Number Guessing Game — a simple command-line application where the program picks a secret random number between 1 and 100, the player guesses, and the program responds with 'Too high', 'Too low', or 'Correct!'. The game reports how many attempts it took to win.

This project is intentionally tiny. The goal is to practise the SDD workflow end-to-end, not to build a complex application.

---

# Prerequisites

Before starting, make sure the following tools are installed on your machine:

| Tool | Notes |
|---|---|
| Python 3.14+ | Download from python.org or install via your system package manager. |
| uv | Python package manager by Astral. Used to create and run the project. See install commands below. |
| Git | Required by specify init to initialise a repository. |
| An AI agent | Claude Code is recommended for this workshop. Others: GitHub Copilot, Gemini CLI, Cursor. |

## Installing uv

uv is not included in any operating system and must be installed separately before you can install specify.

macOS / Linux:

```
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Windows (PowerShell):

```
irm https://astral.sh/uv/install.ps1 | iex
```

## Installing specify

Once uv is installed, install the specify CLI by pinning to the latest stable release (replace vX.Y.Z with the current tag from github.com/github/spec-kit/releases):

```
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@vX.Y.Z
```

Or install from main (may include unreleased changes):

```
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

## Verification commands

> **💡 Tip — verify your tools before the workshop starts**
> python --version       (expect: Python 3.14 or higher)
> uv --version           (expect: uv 0.x.x...)
> specify --version      (expect: Spec Kit version number)
> git --version          (expect: git version 2.x...)

---

# Part 1 — Initialising the Project

## Step 1: Create a Python project with uv

Open a terminal and run the following to create a new Python project:

```
mkdir number-guessing-game
cd number-guessing-game
uv init --app --python 3.14
mkdir tests
touch tests/__init__.py
touch tests/test_main.py
```

> **💡 What each flag does**
> --app       → creates an application project (as opposed to a library).
> --python    → pins the minimum Python version for the project.

uv will produce this structure:

```
number-guessing-game/
├── .python-version
├── pyproject.toml
├── README.md
├── main.py
└── tests/
    ├── __init__.py
    └── test_main.py
```

## Step 2: Initialise Spec Kit

From inside your project directory, run specify init targeting Claude Code (or replace 'claude' with your agent of choice):

```
specify init . --integration claude
```

specify will ask which integration to use if you do not pass --integration. It then copies template files and slash command definitions into your project. You should see a new .specify directory and a CLAUDE.md file (or equivalent for your agent).

> **⚠️ Deprecation notice**
> The --ai flag is deprecated and will be removed in version 1.0.0 or later.
> Always use --integration instead:   specify init . --integration claude

After initialisation your project tree will look like this:

```
number-guessing-game/
├── .specify/
│   ├── memory/
│   │   └── constitution.md        ← created in Part 2
│   ├── scripts/
│   │   ├── create-new-feature.sh
│   │   └── ...other helper scripts
│   ├── specs/                     ← your features go here
│   └── templates/
│       ├── spec-template.md
│       ├── plan-template.md
│       └── tasks-template.md
├── CLAUDE.md                      ← read by Claude Code automatically
├── pyproject.toml
├── main.py
└── tests/
    ├── __init__.py
    └── test_main.py
```

> **💡 What specify actually does**
> The specify CLI copies template files and slash command definitions into your project.
> It does NOT validate your code. It does NOT run tests.
> Once initialised, all further work happens through your AI agent's chat interface.

---

# Part 2 — Establishing Project Principles

The first step in the Spec Kit workflow is establishing a constitution — a set of non-negotiable principles that your AI agent will follow throughout the entire project. Think of it as a rules file that overrides everything else.

## Open your AI agent

Open Claude Code (or your chosen agent) in the project directory. You should see the /speckit-* slash commands available. If you do not, check that specify init completed successfully and re-run it if needed.

## Run /speckit-constitution

In your agent's chat interface, type:

```
/speckit-constitution

Create principles for a beginner Python workshop project.
Focus on: code simplicity over cleverness, meaningful function names,
unit tests for all business logic, no external dependencies beyond
the standard library and pytest, uv as the project manager, Python 3.14+.
```

Your agent will create or update .specify/memory/constitution.md. This file is referenced automatically in every subsequent step. Review it and adjust if anything looks wrong.

> **📝 Why this matters**
> The constitution is the only place to set rules that apply to the whole project.
> For example: if you want the agent to always write tests before implementation (TDD),
> or always use a specific Python version, this is where you declare that.
> Without a constitution, the agent makes its own assumptions.

---

# Part 3 — Writing the Specification

Now you describe what you want to build. The key rule at this stage: describe WHAT and WHY, not HOW. No technology choices yet. No mention of Python frameworks, module names, or implementation details.

## Run /speckit-specify

In your agent's chat interface, type:

```
/speckit-specify

Build a Number Guessing Game for the command line.

The game should:
- Pick a secret number between 1 and 100 at the start of each game.
- Ask the player to enter a guess.
- Tell the player if their guess is too high, too low, or correct.
- Keep asking for guesses until the player guesses correctly.
- At the end, tell the player how many attempts it took.

The game should handle invalid input (non-numbers, numbers outside 1-100)
gracefully, without crashing. It should tell the player what went wrong
and ask them to try again.

The game must be interactively runnable from the terminal: the player
types guesses directly into the running process and receives immediate
feedback without any additional tooling or configuration.
```

Your agent will generate a spec.md file inside a new feature directory under .specify/specs/, for example:

```
.specify/specs/001-number-guessing-game/spec.md
```

The spec.md will contain user stories, functional requirements, and acceptance criteria — all derived from your prompt. Read it carefully. This is your contract for the rest of the workshop.

## Reviewing the spec

Before moving on, check the spec for:

- Are all the behaviours you described present?
- Is there anything in the spec you did not ask for? (Agents can be over-eager.)
- Are the acceptance criteria testable — i.e. can you tell whether they pass or fail?

If anything is missing or wrong, tell your agent directly in plain language and ask it to update spec.md. You do not need to use a slash command for this — free-form feedback works fine at this stage.

> **⚠️ Do not skip the review**
> Everything downstream — the plan, the tasks, and the implementation — is derived from spec.md.
> Errors in the spec will propagate through all subsequent steps.
> This review is the cheapest point to fix mistakes.

---

# Part 4 — Clarifying Ambiguities (Optional but Recommended)

Before creating a technical plan, it is worth running a structured clarification pass. This step finds gaps and ambiguities in the spec that could cause rework during implementation.

## Run /speckit-clarify

In your agent's chat interface, type:

```
/speckit-clarify
```

Your agent will ask targeted questions about areas in the spec that are underspecified. For the guessing game, typical questions might include:

- Should the game be replayable (play again after winning)?
- What exactly should happen when the player types 'abc' instead of a number?
- Should guesses of 0 or negative numbers be treated the same as out-of-range numbers?

Answer each question. The agent will record the answers in a Clarifications section of the spec. These answers become part of the binding specification.

> **💡 When to skip this step**
> For a tiny workshop project you may choose to skip /speckit-clarify.
> In a real project, skipping it is a common cause of expensive rework.
> The rule of thumb: if you are unsure about any edge case, run clarify.

---

# Part 5 — Creating the Technical Plan

With the spec reviewed and clarified, you can now introduce the technology choices. This is the ONLY step where you specify the tech stack. The spec itself must remain technology-agnostic.

## Run /speckit-plan

In your agent's chat interface, type:

```
/speckit-plan

Build this using Python 3.14+ with a uv project structure.
The application is a command-line tool with no external dependencies
beyond the Python standard library.
The entry point must support interactive stdin so the game is fully
playable via 'uv run python main.py'.
Use pytest for unit tests.
Entry point: main.py with a main() function.
Keep the design simple: one module per responsibility.
```

Your agent will produce several planning artifacts in the feature directory:

```
.specify/specs/001-number-guessing-game/
├── spec.md          ← from Part 3
├── plan.md          ← overall technical plan
├── data-model.md    ← class design and data structures
├── research.md      ← notes on library/API choices
└── quickstart.md    ← how to run the project
```

> **💡 How stdin gets handled**
> Because you explicitly mentioned interactive stdin via 'uv run python main.py' in the plan prompt,
> the agent will implement the entry point using standard input() calls.
> You do not need to configure anything manually.
> Verify it appears in plan.md before running /speckit-tasks.

Review plan.md and data-model.md. Key things to check:

- Does the module design match your constitution's simplicity principle?
- Are there any frameworks or dependencies you did not ask for?
- Does the plan cover input validation as described in the spec?

If the agent has added unnecessary complexity, tell it directly and ask it to simplify.

---

# Part 6 — Generating the Task Breakdown

With a validated plan in place, you now break the work into small, ordered, actionable tasks that the agent can execute one by one.

## Run /speckit-tasks

In your agent's chat interface, type:

```
/speckit-tasks
```

Your agent will generate a tasks.md file:

```
.specify/specs/001-number-guessing-game/tasks.md
```

A well-generated tasks.md for the guessing game should contain tasks grouped by user story, for example:

```
## User Story 1: Secret number generation
- [ ] T001: Create number_generator module with generate() function
- [ ] T002: Write pytest tests for number_generator

## User Story 2: Hint feedback
- [ ] T003: Create hint_generator module with generate_hint(secret, guess)
- [ ] T004: Write pytest tests for hint_generator — too high, too low, correct
- [ ] T005: Write pytest tests for boundary values (1, 100)

## User Story 3: Game session
- [ ] T006: Create game_session module tracking attempts and game state
- [ ] T007: Write pytest tests for game_session

## User Story 4: Input validation
- [ ] T008: Create input_validator module
- [ ] T009: Write pytest tests for invalid input cases

## User Story 5: Game loop
- [ ] T010: Implement main game loop in main.py
- [ ] T011: Manual end-to-end test
```

Review tasks.md before proceeding. Verify that every acceptance criterion in spec.md is covered by at least one task. If something is missing, tell your agent and ask it to add the missing tasks.

> **📝 Tasks are the last checkpoint before code is written**
> Once you run /speckit-implement, the agent starts executing tasks.
> Changes after implementation begins are more expensive.
> Take your time reviewing tasks.md — it is worth it.

---

# Part 7 — Implementation

With the spec, plan, and tasks all reviewed and validated, you can now instruct your AI agent to build the project.

## Run /speckit-implement

In your agent's chat interface, type:

```
/speckit-implement
```

Your agent will validate that all required artifacts exist (constitution, spec, plan, tasks), then execute the tasks in order. It will write Python source files, update pyproject.toml if needed, and write pytest tests.

## What to expect during implementation

- The agent executes tasks sequentially, marking each [x] as it completes it.
- It may run uv commands (uv run pytest, uv run python main.py) to verify its work.
- It will ask you questions if it encounters ambiguities not covered by the spec.
- If a task fails, it will attempt to fix the issue before continuing.

> **⚠️ Make sure the required tools are installed**
> The agent will run local CLI commands: python, uv, pytest, etc.
> If Python 3.14+ or uv are not installed, the agent will fail during execution.
> Run 'python --version' and 'uv --version' before starting.

## After implementation

Once the agent completes all tasks, run the tests yourself to confirm everything passes:

```
uv run pytest
```

Then run the game:

```
uv run python main.py
```

Play a full game. If you find a bug or a missing behaviour, check the spec to see if it was covered. If it was covered and the agent missed it, report it back to the agent with a reference to the spec section. If it was not covered, that is a gap in your original spec — a good learning moment.

---

# Part 8 — Reflection and Exercises

## What just happened

You did not write a single line of Python manually. Instead, you:

1. Defined the rules your agent must follow (constitution).
2. Described what to build, without tech details (specify).
3. Chose the tech stack and reviewed the resulting design (plan).
4. Reviewed an ordered task list before any code was written (tasks).
5. Instructed the agent to execute the tasks (implement).

The core discipline is the separation between WHAT (spec) and HOW (plan + tasks). When these are kept separate, you can change the technology stack without changing the spec. You can also review and approve each step before committing to the next.

## Common mistakes to avoid

| Mistake | Why it matters |
|---|---|
| Putting tech details in /speckit-specify | The spec must be technology-agnostic. Tech belongs in /speckit-plan. |
| Skipping the spec review | Errors in spec.md propagate through every subsequent step. |
| Skipping the tasks review | Tasks.md is the last cheap checkpoint before code is written. |
| Treating the first attempt as final | Spec Kit is iterative. Push back on the agent when output is wrong. |
| Not reading constitution.md | The agent follows it. If you have not read it, you may not notice when it is violated. |

## Exercises

### Exercise 1 — Difficulty levels (Easy)

Add a new specification for difficulty levels. Run /speckit-specify again with: 'Add three difficulty levels: Easy (1–50), Medium (1–100, current), Hard (1–200). The player chooses at the start.' Then run /speckit-plan, /speckit-tasks, and /speckit-implement for this new feature. In /speckit-plan, specify Python 3.14+ and pytest as before.

### Exercise 2 — Replay (Medium)

Specify a 'play again' feature. After a win, the game should ask the player if they want to play again. If yes, start a new game. If no, show the player's best score (fewest attempts across all games in the session) and exit.

### Exercise 3 — Audit the spec (Hard)

Read spec.md carefully and find at least two edge cases that are NOT covered. Add them to the spec manually, then run /speckit-tasks again and verify the new tasks cover your additions. Discuss: what would have happened during implementation if those edge cases had not been specified?

---

# Summary

| Command | What it does |
|---|---|
| specify init | Bootstraps the project once. Copies templates and slash commands. |
| /speckit-constitution | Sets binding rules for the AI agent across the whole project. |
| /speckit-specify | Defines what to build. Technology-agnostic. Produces spec.md. |
| /speckit-clarify | Resolves ambiguities before planning. Reduces rework. |
| /speckit-plan | Introduces the tech stack. Produces plan.md, data-model.md, research.md. |
| /speckit-tasks | Breaks the plan into ordered tasks. Produces tasks.md. |
| /speckit-implement | Executes the tasks. Writes code, tests, and build configuration. |

> **📚 Further reading**
> GitHub Spec Kit repository:  github.com/github/spec-kit
> Installation guide:          github.com/github/spec-kit/blob/main/docs/installation.md
> Full SDD methodology:        github.com/github/spec-kit/blob/main/spec-driven.md
> Community walkthroughs:      see the README on the repository
