# SPEC-DRIVEN DEVELOPMENT WORKSHOP

## Building a Number Guessing Game Web Application with [GitHub Spec Kit](https://github.com/github/spec-kit)

### Quarkus + Vanilla HTML/JS Edition

**AI Tool:** Google Gemini / GitHub Copilot

*A beginner-friendly, hands-on workshop for learning spec-first development from scratch using the real [GitHub Spec Kit](https://github.com/github/spec-kit) workflow.*

---

# Introduction

Welcome to this spec-driven development workshop! You will learn how to build software by writing specifications first, then using an AI coding agent to turn those specs into working code. This approach — called Spec-Driven Development (SDD) — helps you think clearly about what your software should do before any code is written.

## What is GitHub Spec Kit?

[GitHub Spec Kit](https://github.com/github/spec-kit) is an open-source toolkit that structures how you work with an AI coding agent. It is NOT a test runner or a code validator. It works by bootstrapping a set of template files and slash commands into your project, which your AI agent then reads and follows to guide you through a structured SDD workflow.

The specify CLI (the command-line tool included in Spec Kit) does one thing: it copies the right template files and slash commands into your project folder for the AI agent you have chosen. After that, all interaction happens through your AI agent.

## The real Spec Kit workflow

| Command | Purpose |
| --- | --- |
| Step 0 | `specify init` — bootstrap the project (run once) |
| Step 1 | `/speckit.constitution` — define project principles |
| Step 2 | `/speckit.specify` — describe what to build (no tech details yet) |
| Step 3 | `/speckit.clarify` — optional: refine ambiguities before planning |
| Step 4 | `/speckit.plan` — choose the tech stack and architecture |
| Step 5 | `/speckit.tasks` — break the plan into actionable tasks |
| Step 6 | `/speckit.implement` — have the AI agent execute the tasks |

> **⚠️ Command naming differs by AI agent**
> Claude Code uses hyphen notation: `/speckit-constitution`, `/speckit-specify`, `/speckit-plan` ...
> Google Gemini / GitHub Copilot use dot notation: `/speckit.constitution`, `/speckit.specify` ...
> This workshop targets Google Gemini / GitHub Copilot. All commands shown use dot notation.
> These commands are typed into the agent's chat interface — NOT into the terminal.
> The `specify` CLI (terminal) is only used once, to initialise the project.

## What you will build

You will build a Number Guessing Game — a web application where the backend picks a secret random number between 1 and 100, the player submits guesses through a browser page, and the application responds with "Too high", "Too low", or "Correct!". The game reports how many attempts it took to win.

The backend is built with **Quarkus** (Java), exposing a small REST API. The frontend is a **single HTML file** with vanilla JavaScript — no build tools, no frameworks, no npm.

This project is intentionally small. The goal is to practise the SDD workflow end-to-end, not to build a complex application.

---

# Prerequisites

Before starting, make sure the following tools are installed on your machine:

| Tool | Notes |
| --- | --- |
| Java JDK 21 | LTS release. Quarkus 3.x requires Java 17+; Java 21 LTS is recommended. Any distribution works: OpenJDK, Temurin, Amazon Corretto. |
| Maven 3.9+ | Used by the Quarkus project. The Maven Wrapper (`mvnw`) bundled by Quarkus can be used — no global install strictly required. |
| Python 3.11+ | Required by the specify CLI. Download from python.org. |
| uv | Python package manager by Astral. Used to install specify. See install commands below. |
| Git | Required by `specify init` to initialise a repository. |
| An AI agent | Google Gemini is recommended for this workshop. Others: GitHub Copilot, Claude Code, Cursor. |
| A modern browser | Chrome, Firefox, or Safari. Required to test the frontend. |

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

Once uv is installed, install the specify CLI by pinning to the latest stable release (replace `vX.Y.Z` with the current tag from github.com/github/spec-kit/releases):

```
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@vX.Y.Z
```

Or install from main (may include unreleased changes):

```
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
```

## Verification commands

> **💡 Tip — verify your tools before the workshop starts**
> `java -version` (expect: openjdk 21...)
> `mvn --version` (expect: Apache Maven 3.9...)
> `python --version` (expect: Python 3.11 or higher)
> `uv --version` (expect: uv 0.x.x...)
> `specify --version` (expect: Spec Kit version number)
> `git --version` (expect: git version 2.x...)

---

# Part 1 — Initialising the Project

## Step 1: Create a Quarkus project

Open a terminal and use the Quarkus CLI or the Maven plugin to scaffold a new project. The Maven approach requires no additional tools beyond what is already installed:

```
mvn io.quarkus.platform:quarkus-maven-plugin:3.21.1:create \
  -DprojectGroupId=com.workshop.game \
  -DprojectArtifactId=number-guessing-game \
  -DprojectVersion=1.0.0-SNAPSHOT \
  -Dextensions="rest-jackson" \
  -DnoCode
```

> **💡 What each parameter does**
> `extensions="rest-jackson"` → adds the Quarkus REST (Jakarta REST) and Jackson JSON extensions, which are all this project needs.
> `-DnoCode` → skips the default generated example resource and test, giving you a clean project.
> The command contacts the Quarkus Maven repository to download the BOM and extensions. An internet connection is required.

Move into the project directory:

```
cd number-guessing-game
```

Quarkus will produce this structure:

```
number-guessing-game/
├── src/
│   ├── main/
│   │   ├── java/com/workshop/game/    ← your Java source files go here
│   │   └── resources/
│   │       ├── application.properties
│   │       └── META-INF/resources/    ← static files served by Quarkus (index.html goes here)
│   └── test/
│       └── java/com/workshop/game/    ← your JUnit tests go here
├── mvnw
├── mvnw.cmd
└── pom.xml
```

> **💡 Where to put the frontend**
> Quarkus automatically serves any file placed in `src/main/resources/META-INF/resources/` at the root URL.
> Put `index.html` there and it will be available at `http://localhost:8080/`.
> No separate web server or proxy is needed.

## Step 2: Initialise Spec Kit

From inside your project directory, run `specify init` targeting Google Gemini (or replace `gemini` with your agent of choice):

```
specify init . --integration gemini
```

`specify` will ask which integration to use if you do not pass `--integration`. It then copies template files and slash command definitions into your project. You should see a new `.specify` directory and a `.google-gemini-instructions.md` file (or equivalent for your agent).

> **⚠️ Deprecation notice**
> The `--ai` flag is deprecated and will be removed in version 1.0.0 or later.
> Always use `--integration` instead: `specify init . --integration gemini`

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
├── GEMINI.md        ← read by Google Gemini automatically
├── src/
│   └── ...Quarkus source tree
├── mvnw
└── pom.xml
```

> **💡 What specify actually does**
> The specify CLI copies template files and slash command definitions into your project.
> It does NOT validate your code. It does NOT run tests.
> Once initialised, all further work happens through your AI agent's chat interface.

---

# Part 2 — Establishing Project Principles

The first step in the Spec Kit workflow is establishing a constitution — a set of non-negotiable principles that your AI agent will follow throughout the entire project. Think of it as a rules file that overrides everything else.

## Open your AI agent

Open Google Gemini (or your chosen agent) in the project directory. You should see the `/speckit.*` slash commands available. If you do not, check that `specify init` completed successfully and re-run it if needed.

## Run /speckit.constitution

In your agent's chat interface, type:

```
/speckit.constitution

Create principles for a beginner Java web application workshop project.
Focus on: code simplicity over cleverness, meaningful method names,
unit tests for all business logic, no external dependencies beyond
the JDK, Quarkus, and JUnit 5, Maven as the build tool, Java 21 LTS.
The frontend must be a single index.html file using only vanilla
HTML and JavaScript — no npm, no bundlers, no frontend frameworks.
```

Your agent will create or update `.specify/memory/constitution.md`. This file is referenced automatically in every subsequent step. Review it and adjust if anything looks wrong.

> **📝 Why this matters**
> The constitution is the only place to set rules that apply to the whole project.
> For example: if you want the agent to always write tests before implementation (TDD),
> or to always keep the frontend dependency-free, this is where you declare that.
> Without a constitution, the agent makes its own assumptions — and may introduce
> a React build pipeline when you asked for a single HTML file.

---

# Part 3 — Writing the Specification

Now you describe what you want to build. The key rule at this stage: describe **WHAT** and **WHY**, not **HOW**. No technology choices yet. No mention of Java, Quarkus, REST, or HTML.

## Run /speckit.specify

In your agent's chat interface, type:

```
/speckit.specify

Build a Number Guessing Game accessible through a web browser.

The game should:
- Pick a secret number between 1 and 100 at the start of each game.
- Allow the player to submit a guess through a web page.
- Tell the player if their guess is too high, too low, or correct.
- Keep track of how many attempts the player has made.
- At the end, tell the player how many attempts it took to win.
- Allow the player to start a new game after winning.

The game should handle invalid input (non-numbers, numbers outside 1-100)
gracefully, without crashing. It should tell the player what went wrong
and allow them to try again.

Each game session must be independent: two players using the browser
at the same time must each have their own secret number and attempt count.
```

Your agent will generate a `spec.md` file inside a new feature directory under `.specify/specs/`, for example:

```
.specify/specs/001-number-guessing-game/spec.md
```

The `spec.md` will contain user stories, functional requirements, and acceptance criteria — all derived from your prompt. Read it carefully. This is your contract for the rest of the workshop.

## Reviewing the spec

Before moving on, check the spec for:

- Are all the behaviours you described present?
- Is there anything in the spec you did not ask for? (Agents can be over-eager.)
- Are the acceptance criteria testable — i.e. can you tell whether they pass or fail?

If anything is missing or wrong, tell your agent directly in plain language and ask it to update `spec.md`. You do not need to use a slash command for this — free-form feedback works fine at this stage.

> **⚠️ Do not skip the review**
> Everything downstream — the plan, the tasks, and the implementation — is derived from `spec.md`.
> Errors in the spec will propagate through all subsequent steps.
> This review is the cheapest point to fix mistakes.

---

# Part 4 — Clarifying Ambiguities (Optional but Recommended)

Before creating a technical plan, it is worth running a structured clarification pass. This step finds gaps and ambiguities in the spec that could cause rework during implementation.

## Run /speckit.clarify

In your agent's chat interface, type:

```
/speckit.clarify
```

Your agent will ask targeted questions about areas in the spec that are underspecified. For the guessing game, typical questions might include:

- Should each browser tab get its own independent game session, or is session shared across tabs?
- What should happen if the player navigates away and comes back — does the game reset?
- Should the attempt count be shown during the game, or only revealed at the end?

Answer each question. The agent will record the answers in a Clarifications section of the spec. These answers become part of the binding specification.

> **💡 When to skip this step**
> For a tiny workshop project you may choose to skip `/speckit.clarify`.
> In a real project, skipping it is a common cause of expensive rework.
> The rule of thumb: if you are unsure about any edge case, run clarify.

---

# Part 5 — Creating the Technical Plan

With the spec reviewed and clarified, you can now introduce the technology choices. This is the **ONLY** step where you specify the tech stack. The spec itself must remain technology-agnostic.

## Run /speckit.plan

In your agent's chat interface, type:

```
/speckit.plan

Build this using Java 21 with Quarkus 3.x as the backend framework.
Expose a small REST API with two endpoints:
  POST /api/game        → start a new game, returns a session ID
  POST /api/game/guess  → submit a guess, returns the hint and attempt count

Game state (secret number, attempt count) must be stored server-side,
keyed by session ID. Use a simple in-memory map — no database required.
Generate session IDs with java.util.UUID.

The frontend is a single file: src/main/resources/META-INF/resources/index.html
It must use only vanilla HTML and JavaScript — no npm, no bundlers, no frameworks.
The page communicates with the backend using the Fetch API.

Use JUnit 5 for unit tests. Test business logic classes directly — no HTTP-level tests required.
Package: com.workshop.game
Keep the design simple: one class per responsibility.
```

Your agent will produce several planning artifacts in the feature directory:

```
.specify/specs/001-number-guessing-game/
├── spec.md          ← from Part 3
├── plan.md          ← overall technical plan
├── data-model.md    ← class design and REST API contract
├── research.md      ← notes on Quarkus extension choices
└── quickstart.md    ← how to run the project
```

> **💡 Key design decisions to verify in plan.md**
> - The REST API contract: endpoint paths, request/response JSON shapes, and HTTP status codes.
> - Session management: the in-memory map approach and its lifecycle (e.g. does the map grow forever or is there a cleanup strategy?).
> - The `index.html` approach: confirm no external JS dependencies are listed.

Review `plan.md` and `data-model.md`. Key things to check:

- Does the class design match your constitution's simplicity principle?
- Are there any frameworks or dependencies you did not ask for?
- Does the plan cover input validation as described in the spec?
- Does the REST API contract look reasonable — sensible status codes, clear JSON fields?

If the agent has added unnecessary complexity (e.g. introduced a database, a JS framework, or a separate frontend build step), tell it directly and ask it to simplify.

---

# Part 6 — Generating the Task Breakdown

With a validated plan in place, you now break the work into small, ordered, actionable tasks that the agent can execute one by one.

## Run /speckit.tasks

In your agent's chat interface, type:

```
/speckit.tasks
```

Your agent will generate a `tasks.md` file:

```
.specify/specs/001-number-guessing-game/tasks.md
```

A well-generated `tasks.md` for the guessing game should contain tasks grouped by user story, for example:

```
## User Story 1: Secret number generation
- [ ] T001: Create NumberGenerator class with generate() method
- [ ] T002: Write unit tests for NumberGenerator

## User Story 2: Hint feedback
- [ ] T003: Create HintGenerator class with hint(secret, guess) method
- [ ] T004: Write unit tests for HintGenerator — too high, too low, correct
- [ ] T005: Write unit tests for boundary values (1, 100)

## User Story 3: Game session management
- [ ] T006: Create GameSession class (sessionId, secretNumber, attemptCount)
- [ ] T007: Create GameService class managing sessions in an in-memory map
- [ ] T008: Write unit tests for GameService

## User Story 4: REST API
- [ ] T009: Create GameResource (Jakarta REST) with POST /api/game
- [ ] T010: Add POST /api/game/guess to GameResource
- [ ] T011: Write unit tests for GameResource using QuarkusTest

## User Story 5: Input validation
- [ ] T012: Create InputValidator class
- [ ] T013: Write unit tests for invalid input cases

## User Story 6: Frontend
- [ ] T014: Create index.html with game UI (input, submit button, message area)
- [ ] T015: Add JavaScript to call POST /api/game on page load
- [ ] T016: Add JavaScript to call POST /api/game/guess on submit
- [ ] T017: Manual end-to-end test in browser
```

Review `tasks.md` before proceeding. Verify that every acceptance criterion in `spec.md` is covered by at least one task. If something is missing, tell your agent and ask it to add the missing tasks.

> **📝 Tasks are the last checkpoint before code is written**
> Once you run `/speckit.implement`, the agent starts executing tasks.
> Changes after implementation begins are more expensive.
> Take your time reviewing `tasks.md` — it is worth it.

---

# Part 7 — Implementation

With the spec, plan, and tasks all reviewed and validated, you can now instruct your AI agent to build the project.

## Run /speckit.implement

In your agent's chat interface, type:

```
/speckit.implement
```

Your agent will validate that all required artifacts exist (constitution, spec, plan, tasks), then execute the tasks in order. It will write Java source files, update `pom.xml` if needed, and write JUnit tests. It will also create `index.html`.

## What to expect during implementation

- The agent executes tasks sequentially, marking each `[x]` as it completes it.
- It may run Maven commands (`./mvnw compile`, `./mvnw test`) to verify its work.
- It will ask you questions if it encounters ambiguities not covered by the spec.
- If a task fails, it will attempt to fix the issue before continuing.

> **⚠️ Make sure the required tools are installed**
> The agent will run local CLI commands: `java`, `./mvnw`, etc.
> If Java 21 is not installed, the agent will fail during compilation.
> Run `java -version` before starting.

## After implementation

Once the agent completes all tasks, run the tests yourself to confirm everything passes:

```
./mvnw test
```

Then start the application in development mode:

```
./mvnw quarkus:dev
```

Open your browser at `http://localhost:8080`. Play a full game. Quarkus dev mode supports live reload: any changes you make to Java files or `index.html` take effect immediately without restarting.

If you find a bug or a missing behaviour, check the spec to see if it was covered. If it was covered and the agent missed it, report it back to the agent with a reference to the spec section. If it was not covered, that is a gap in your original spec — a good learning moment.

> **💡 Dev mode vs production mode**
> `./mvnw quarkus:dev` — live reload, enhanced error pages, Dev UI at `/q/dev`. Use during development.
> `./mvnw package` then `java -jar target/quarkus-app/quarkus-run.jar` — production build. No live reload.

---

# Part 8 — Reflection and Exercises

## What just happened

You did not write a single line of Java or HTML manually. Instead, you:

1. Defined the rules your agent must follow (constitution).
2. Described what to build, without tech details (specify).
3. Chose the tech stack and reviewed the resulting design (plan).
4. Reviewed an ordered task list before any code was written (tasks).
5. Instructed the agent to execute the tasks (implement).

The core discipline is the separation between **WHAT** (spec) and **HOW** (plan + tasks). When these are kept separate, you can change the technology stack without changing the spec. Notice that your `spec.md` never mentions Quarkus, REST, HTML, or Java — yet the entire implementation follows from it.

## Common mistakes to avoid

| Mistake | Why it matters |
| --- | --- |
| Putting tech details in `/speckit.specify` | The spec must be technology-agnostic. Tech belongs in `/speckit.plan`. |
| Skipping the spec review | Errors in `spec.md` propagate through every subsequent step. |
| Skipping the tasks review | `tasks.md` is the last cheap checkpoint before code is written. |
| Treating the first attempt as final | Spec Kit is iterative. Push back on the agent when output is wrong. |
| Not reading `constitution.md` | The agent follows it. If you have not read it, you may not notice when it is violated. |
| Letting the agent add a JS framework | The constitution blocks this. If it happens anyway, correct the agent and re-check. |

## Exercises

### Exercise 1 — Difficulty levels (Easy)

Add a new specification for difficulty levels. Run `/speckit.specify` again with: 'Add three difficulty levels: Easy (1–50), Medium (1–100, current), Hard (1–200). The player chooses at the start of each game.' Then run `/speckit.plan`, `/speckit.tasks`, and `/speckit.implement` for this new feature.

### Exercise 2 — Game history (Medium)

Specify a 'session history' feature. After each completed game, the result (number of attempts, secret number) should be appended to a visible history list on the page. The history is in-memory only — it resets when the page is refreshed.

### Exercise 3 — Audit the spec (Hard)

Read `spec.md` carefully and find at least two edge cases that are NOT covered. Add them to the spec manually, then run `/speckit.tasks` again and verify the new tasks cover your additions. Consider: what happens if a player submits a guess before starting a game? What happens if the session ID is tampered with?

---

# Summary

| Command | What it does |
| --- | --- |
| `specify init` | Bootstraps the project once. Copies templates and slash commands. |
| `/speckit.constitution` | Sets binding rules for the AI agent across the whole project. |
| `/speckit.specify` | Defines what to build. Technology-agnostic. Produces `spec.md`. |
| `/speckit.clarify` | Resolves ambiguities before planning. Reduces rework. |
| `/speckit.plan` | Introduces the tech stack. Produces `plan.md`, `data-model.md`, `research.md`. |
| `/speckit.tasks` | Breaks the plan into ordered tasks. Produces `tasks.md`. |
| `/speckit.implement` | Executes the tasks. Writes code, tests, and static frontend files. |

> **📚 Further reading**
> GitHub Spec Kit repository: [github.com/github/spec-kit](https://github.com/github/spec-kit)
> Installation guide: [github.com/github/spec-kit/blob/main/docs/installation.md](https://github.com/github/spec-kit/blob/main/docs/installation.md)
> Full SDD methodology: [github.com/github/spec-kit/blob/main/spec-driven.md](https://github.com/github/spec-kit/blob/main/spec-driven.md)
> Quarkus getting started: quarkus.io/get-started
> Quarkus REST extension docs: quarkus.io/guides/rest
