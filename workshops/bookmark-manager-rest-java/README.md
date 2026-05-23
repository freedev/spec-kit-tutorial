# SPEC-DRIVEN DEVELOPMENT WORKSHOP — INTERMEDIATE

## Building a Bookmark Manager REST API with GitHub Spec Kit

### Quarkus 3.31 + Java 25 + Gradle 9 Edition

**AI Tool:** Claude Code

*A hands-on workshop for practising spec-first development on a realistic backend service — REST endpoints, relational persistence, migrations, validation, structured errors, and integration tests.*

---

# Introduction

Welcome to the intermediate Spec-Driven Development workshop. This workshop assumes you have completed the beginner workshop (the Number Guessing Game) or are otherwise familiar with the Spec Kit loop: `constitution → specify → clarify → plan → tasks → implement`. If you are new to Spec Kit, complete the beginner workshop first — the loop is identical, but here the domain is large enough that skipping fundamentals will hurt.

## What is different about this workshop

The beginner workshop teaches you the workflow. This workshop teaches you how the workflow behaves when the domain is **non-trivial**: a real REST API, a real database, real non-functional requirements. You will encounter situations the beginner workshop cannot produce:

- The agent will make architectural choices in `plan.md` that you must evaluate.
- The agent will silently add dependencies and patterns you did not ask for. You will learn to spot and reject them.
- `/speckit-clarify` becomes essential, not optional — a single ambiguity in the spec will cascade into hours of rework.
- The constitution does real work: most "the agent did something I didn't want" moments trace back to a rule that should have been written down.

## What you will build

A **Bookmark Manager REST API**. Users can create, list, retrieve, update, and delete bookmarks; tag bookmarks; filter bookmarks by tag; and search bookmarks by keyword. The data is persisted in PostgreSQL. The API exposes an OpenAPI document and returns RFC 7807 Problem Details on errors.

The scope is intentionally larger than the beginner workshop, but bounded. Two resources, ~7 endpoints, one many-to-many relationship. Enough surface area to expose every interesting aspect of Spec Kit; small enough to complete in a single session.

## What you will NOT build

To keep the workshop tractable, the following are **explicitly out of scope** and must be excluded from your spec:

- Authentication, authorisation, or user accounts.
- Rate limiting, caching, or pagination beyond simple `limit`/`offset`.
- Frontend or UI of any kind.
- Deployment, containerisation, or CI/CD.
- Soft deletes, audit trails, or event sourcing.

If your agent proposes any of these unprompted, push back. They are common over-engineering traps.

## The Spec Kit workflow (recap)

| Command | Purpose |
|---|---|
| Step 0 | `specify init` — bootstrap the project (run once) |
| Step 1 | `/speckit-constitution` — define project principles |
| Step 2 | `/speckit-specify` — describe what to build (no tech details) |
| Step 3 | `/speckit-clarify` — refine ambiguities (mandatory at this scale) |
| Step 4 | `/speckit-plan` — choose the tech stack and architecture |
| Step 5 | `/speckit-tasks` — break the plan into actionable tasks |
| Step 6 | `/speckit-implement` — execute the tasks |

> **⚠️ Command naming differs by AI agent**
> Claude Code uses hyphen notation:   `/speckit-constitution`, `/speckit-specify`, `/speckit-plan` ...
> GitHub Copilot / Gemini CLI use dot notation:  `/speckit.constitution`, `/speckit.specify` ...
> This workshop targets Claude Code. All commands shown use hyphen notation.

## Expected duration

Plan for **3 to 4 hours**. The bulk of the time is in reviewing the agent's output, not waiting for it. If you finish in less than 2 hours, you almost certainly skipped a review step and you will regret it.

---

# Prerequisites

Before starting, make sure the following are installed:

| Tool | Notes |
|---|---|
| Java JDK 25 | LTS release. OpenJDK, Temurin, or Amazon Corretto all work. |
| Gradle 9 | The Quarkus CLI generates a Gradle wrapper, so a global install is optional. |
| Python 3.11+ | Required by the `specify` CLI. |
| uv | Python package manager. Used to install `specify`. |
| Git | Required by `specify init`. |
| Docker (or Podman) | **Required.** Quarkus Dev Services starts a PostgreSQL container automatically. |
| Quarkus CLI | Primary way to scaffold the project. Install instructions below. |
| An AI agent | Claude Code is recommended. Others: GitHub Copilot, Gemini CLI, Cursor. |

## Installing uv

```
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows (PowerShell)
irm https://astral.sh/uv/install.ps1 | iex
```

## Installing specify

Pin to a stable release (replace `vX.Y.Z` with the current tag from `github.com/github/spec-kit/releases`):

```
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git@vX.Y.Z
```

## Installing the Quarkus CLI

The recommended path is via JBang or SDKMAN. With SDKMAN:

```
sdk install quarkus
```

With JBang:

```
jbang trust add https://repo1.maven.org/maven2/io/quarkus/quarkus-cli/
jbang app install --fresh --force quarkus@quarkusio
```

> **💡 Fallback if the CLI is unavailable**
> You can scaffold via the web UI at `code.quarkus.io`, then unzip the result and `cd` into it.
> Note that at the time of writing, `code.quarkus.io` may not yet offer Java 25 as a selectable option even for Quarkus 3.31+. If so, generate with Java 21 and bump `sourceCompatibility`/`targetCompatibility` to 25 manually in `build.gradle` before going further.

## Verification commands

```
java -version          (expect: openjdk 25...)
gradle --version       (expect: Gradle 9...)
python --version       (expect: Python 3.11 or higher)
uv --version           (expect: uv 0.x.x...)
specify --version      (expect: Spec Kit version number)
quarkus --version      (expect: 3.31.x or later)
docker --version       (expect: Docker 24+ or compatible)
git --version          (expect: git version 2.x...)
```

> **⚠️ Docker is not optional**
> Quarkus Dev Services launches a PostgreSQL container the first time you run the app or its tests. If Docker (or Podman with the Docker-compatible socket) is not running, the build will fail at the integration test stage.

---

# Part 1 — Initialising the Project

## Step 1: Create a Quarkus project

Open a terminal and scaffold a Quarkus application:

```
quarkus create app com.workshop.bookmarks:bookmarks-api \
  --gradle \
  --java=25 \
  --extension=rest,rest-jackson,hibernate-orm-panache, \
              jdbc-postgresql,flyway,smallrye-openapi, \
              hibernate-validator,logging-json,opentelemetry
cd bookmarks-api
```

> **💡 What each extension does**
> `rest` + `rest-jackson` — Quarkus REST (RESTEasy Reactive) and JSON serialisation.
> `hibernate-orm-panache` — Hibernate ORM with Panache repositories.
> `jdbc-postgresql` — PostgreSQL JDBC driver.
> `flyway` — versioned schema migrations.
> `smallrye-openapi` — auto-generated OpenAPI document and Swagger UI in dev mode.
> `hibernate-validator` — Jakarta Bean Validation (`@NotNull`, `@Size`, etc.).
> `logging-json` — structured (JSON) log output.
> `opentelemetry` — distributed tracing and automatic trace IDs in the log MDC.

The generated structure should look like:

```
bookmarks-api/
├── build.gradle
├── gradle/
├── gradlew
├── settings.gradle
└── src/
    ├── main/
    │   ├── java/com/workshop/bookmarks/
    │   └── resources/
    │       └── application.properties
    └── test/
        └── java/com/workshop/bookmarks/
```

## Step 2: Verify the project runs

Before introducing Spec Kit, confirm the empty Quarkus project starts:

```
./gradlew quarkusDev
```

You should see Quarkus start in dev mode and listen on port 8080. The first start may pull container images for Dev Services — give it a minute. Press `q` to quit.

If this step fails, **fix it before continuing**. Spec Kit cannot help you debug a broken project skeleton.

## Step 3: Initialise Spec Kit

From inside the project directory:

```
specify init . --integration claude
```

After initialisation your project tree will include:

```
bookmarks-api/
├── .specify/
│   ├── memory/
│   │   └── constitution.md        ← created in Part 2
│   ├── scripts/
│   ├── specs/                     ← features go here
│   └── templates/
│       ├── spec-template.md
│       ├── plan-template.md
│       └── tasks-template.md
├── CLAUDE.md                      ← read by Claude Code automatically
├── build.gradle
├── settings.gradle
└── src/
```

> **⚠️ Deprecation notice**
> The `--ai` flag is deprecated and will be removed in version 1.0.0 or later.
> Always use `--integration` instead.

---

# Part 2 — Establishing Project Principles

The constitution is the highest-leverage file in this workshop. At the beginner-workshop scale, a weak constitution costs you a few minutes of cleanup. At this scale, a weak constitution costs you an entire afternoon of fighting the agent over Lombok, MapStruct, generic exception handlers, and three different ways of writing repositories.

## Run /speckit-constitution

In your agent's chat interface, type:

```
/speckit-constitution

Create principles for a Quarkus 3.31+ REST API project. The constitution must
be specific and enforceable. Include the following non-negotiable rules:

CODE STYLE
- Java 25 syntax and language features are permitted.
- No Lombok. Use Java records for DTOs and plain classes with explicit
  constructors for entities.
- No MapStruct. Mapping between entities and DTOs is done in explicit static
  factory methods on the DTO records.
- One class per file. One responsibility per class.
- Package layout: resource, service, repository, domain, dto, exception.
  Each layer talks only to the layer below it.

PERSISTENCE
- Hibernate ORM with classic Panache, repository style (PanacheRepository).
  Do not use the active-record (PanacheEntity) style.
- All schema changes go through Flyway migrations in
  src/main/resources/db/migration. The migration filename pattern is V<n>__<description>.sql.
- quarkus.hibernate-orm.database.generation must be set to 'validate' in
  application.properties. Never 'update', 'create', or 'drop-and-create'
  in any profile.

REST AND ERROR HANDLING
- All request bodies are DTO records annotated with Jakarta Validation
  constraints. Resource methods always use @Valid on incoming bodies.
- All errors are returned as RFC 7807 Problem Details (application/problem+json).
- A central ExceptionMapper handles validation errors, constraint violations,
  not-found errors, and uncaught exceptions. No try/catch inside resource
  classes for the purpose of converting exceptions to HTTP responses.

API DOCUMENTATION
- OpenAPI is auto-generated via smallrye-openapi.
- Every resource method has an @Operation summary and at least one
  @APIResponse describing the success case.

OBSERVABILITY
- Logging uses SLF4J. Log output is JSON (quarkus-logging-json).
- OpenTelemetry is enabled. Trace IDs appear in log MDC automatically.
- Never log request bodies or full URLs that may contain user data.
  Log entity IDs and outcome status only.

TESTING
- Unit tests for the service layer use plain JUnit 6 and Mockito.
- Integration tests use @QuarkusTest and RestAssured. They rely on Quarkus
  Dev Services to provide PostgreSQL — do not configure Testcontainers manually.
- Every endpoint has at least one happy-path integration test and one
  failure-mode integration test (validation error, not found, or conflict).
- Every service method has unit tests for its primary success path and
  for each business-rule failure.

DEPENDENCIES
- No new dependencies may be added without an explicit request from the
  developer. If the agent believes a dependency is required, it must ask
  before adding it.

PROCESS
- Tests are written alongside (not after) the code they cover. Each task
  in tasks.md must produce both production code and its tests, or be a
  pure-test task that follows a paired production-code task.
```

Your agent will create or update `.specify/memory/constitution.md`. **Read it line by line.** Every rule above must be present, in some form, in the generated constitution. If anything is missing, garbled, or contradicted, fix it now — either by editing the file directly or by telling the agent to revise it.

> **📝 Why this matters**
> The constitution is referenced automatically at every subsequent step. Most "the agent did X and I didn't want it" problems trace back to a missing line here. The constitution above is comprehensive on purpose. In a real project, you would build it up over time; in a workshop, you cannot afford to.

> **⚠️ Common agent additions to push back on**
> - Lombok, MapStruct, ModelMapper — explicitly forbidden by the constitution above.
> - Spring annotations (`@RestController`, `@Service`) — this is Quarkus, not Spring.
> - `quarkus.hibernate-orm.database.generation=update` — forbidden; we use Flyway.
> - A generic `RuntimeException`-catching mapper that swallows everything — forbidden by the error-handling rule.

---

# Part 3 — Writing the Specification

Now describe what to build. **No technology choices, no class names, no mention of Quarkus, Panache, or PostgreSQL.** The spec must be technology-agnostic. If you can swap the entire stack later without rewriting `spec.md`, you have written a good spec.

## Run /speckit-specify

In your agent's chat interface, type:

```
/speckit-specify

Build a Bookmark Manager REST API.

DOMAIN
A bookmark represents a saved web link. Each bookmark has:
- a URL (required, must be a valid http or https URL),
- a title (required, 1 to 200 characters),
- an optional description (up to 2000 characters),
- a creation timestamp (set by the system, never accepted from clients),
- a last-updated timestamp (set by the system on every update),
- zero or more tags.

A tag is a short label that can be attached to many bookmarks, and a bookmark
may have many tags. Each tag has:
- a name (required, 1 to 50 characters, lowercase letters, digits, and hyphens only),
- a creation timestamp.

Tag names are unique across the system. A bookmark cannot have the same tag
attached twice.

ENDPOINTS

The API exposes the following capabilities:

1. Create a bookmark. The client provides URL, title, optional description,
   and an optional list of tag names. Any tag name that does not already
   exist is created on the fly. Returns the created bookmark including its
   server-assigned ID and timestamps.

2. List bookmarks. Supports optional filtering by a single tag name and
   optional keyword search across title and description. Supports
   pagination via 'limit' (default 20, max 100) and 'offset' (default 0)
   query parameters. Results are ordered by creation timestamp, newest first.

3. Get a single bookmark by ID, including its tags.

4. Update a bookmark by ID. The client may update title, description, URL,
   and the full set of tags. Tags provided in the update replace the existing
   tag set entirely (this is a PUT, not a PATCH). Creation timestamp is
   never modified.

5. Delete a bookmark by ID. Deleting a bookmark does not delete its tags.
   Tags that become orphaned (attached to no bookmark) are NOT automatically
   deleted in this version.

6. List all tags, including for each tag the count of bookmarks currently
   using it. Results are ordered alphabetically.

7. Delete a tag by name. Deleting a tag removes it from every bookmark that
   uses it. The bookmarks themselves are not deleted.

ERROR BEHAVIOUR

The API must respond with appropriate HTTP status codes and a structured
error body (Problem Details, RFC 7807) for:
- malformed JSON requests,
- validation failures (invalid URL, title too long, tag name with disallowed
  characters, etc.),
- attempts to fetch, update, or delete a resource that does not exist,
- attempts to use a tag name format that violates the naming rules,
- any unexpected server-side error.

Validation errors must include enough detail for a client to identify which
field failed and why.

OUT OF SCOPE

The following are explicitly NOT part of this specification:
- Authentication, authorisation, or per-user data.
- Soft delete or audit history.
- Caching, rate limiting, or background jobs.
- Bulk operations (e.g. delete-many).
- Webhook or event notifications.
- Pagination beyond simple limit/offset (no cursors, no total-count headers).
```

Your agent will generate `spec.md` inside a new feature directory, for example:

```
.specify/specs/001-bookmark-manager/spec.md
```

## Reviewing the spec

This review is the most important checkpoint in the workshop. Read the spec slowly. Verify:

- **Every endpoint** you described is present, with a clear statement of inputs, outputs, and failure modes.
- **No technology** appears in the spec. If you see "Hibernate", "PanacheRepository", "Quarkus", "PostgreSQL", or any Java type, that is a defect — make the agent remove it.
- **Acceptance criteria are testable.** "Returns the created bookmark" is not testable. "Returns HTTP 201 with a body containing the assigned id, the submitted title, and the submitted URL" is testable.
- **The out-of-scope section is honoured.** The agent should not have added authentication, soft deletes, or pagination cursors. If it did, instruct it to remove them and re-generate.

> **⚠️ Do not skip the review**
> Everything downstream — clarifications, the plan, the tasks, the implementation — is derived from `spec.md`. A defect here propagates everywhere.

---

# Part 4 — Clarifying Ambiguities (Mandatory)

In the beginner workshop, `/speckit-clarify` was optional. At this scale, it is mandatory. A small REST API will produce 5–15 genuine clarification questions, and skipping them guarantees rework.

## Run /speckit-clarify

In your agent's chat interface, type:

```
/speckit-clarify
```

Your agent will ask targeted questions. Expect questions in roughly these areas — answer each one explicitly, even if the answer feels obvious:

**URL validation**
- Must URLs use `http` or `https` exclusively, or are other schemes allowed?
  → *Answer: `http` and `https` only.*
- Are duplicate URLs allowed (two bookmarks with the same URL)?
  → *Answer: yes, duplicates are allowed.*

**Tag handling**
- When creating a bookmark with a tag that already exists, is the existing tag reused or is the request rejected?
  → *Answer: existing tag is reused, case-insensitive match on name.*
- If a tag name is provided in mixed case (e.g. `MyTag`), is it normalised to lowercase or rejected?
  → *Answer: rejected, with a validation error pointing to the tag name.*

**Update semantics**
- On a `PUT /bookmarks/{id}`, if the `tags` field is absent from the body, does that mean "leave tags unchanged" or "remove all tags"?
  → *Answer: absent means "remove all tags". This is a `PUT`, not a `PATCH`.*

**Pagination edge cases**
- What does the response look like when `offset` is past the end of the dataset?
  → *Answer: HTTP 200 with an empty list.*
- What happens if `limit=0`?
  → *Answer: validation error.*

**Search behaviour**
- Is keyword search case-sensitive?
  → *Answer: case-insensitive.*
- Does it match substrings, whole words, or prefixes?
  → *Answer: substring match.*

**Concurrency**
- If two clients update the same bookmark simultaneously, is last-write-wins acceptable, or must we detect the conflict?
  → *Answer: last-write-wins is acceptable for this version.*

The agent will record your answers in a Clarifications section appended to `spec.md`. These answers are now binding.

> **💡 Why this step pays off**
> Every question above corresponds to a decision the agent would otherwise have made silently — and probably differently from what you wanted. Catching them here costs minutes. Catching them after implementation costs hours.

---

# Part 5 — Creating the Technical Plan

With the spec clarified, you now introduce the technology choices. This is the **only** place where the stack is specified.

## Run /speckit-plan

In your agent's chat interface, type:

```
/speckit-plan

Build this using Quarkus 3.31 or later on Java 25 with a Gradle 9 build.

EXTENSIONS
Use only the extensions already on the classpath:
  quarkus-rest, quarkus-rest-jackson,
  quarkus-hibernate-orm-panache, quarkus-jdbc-postgresql, quarkus-flyway,
  quarkus-smallrye-openapi, quarkus-hibernate-validator,
  quarkus-logging-json, quarkus-opentelemetry.
Do not add any new extension or third-party dependency without asking first.

PERSISTENCE
Use Hibernate ORM with classic Panache in repository style (PanacheRepository).
Do not use PanacheEntity (active-record style). Entities are plain classes
with explicit fields, constructors, getters, and setters where needed —
no @PanacheEntity inheritance.

DATABASE
PostgreSQL via Quarkus Dev Services in dev and test profiles. No manual
Testcontainers configuration. Schema is managed entirely by Flyway:
  src/main/resources/db/migration/V1__init.sql
quarkus.hibernate-orm.database.generation = validate
quarkus.flyway.migrate-at-start = true

PACKAGE LAYOUT
com.workshop.bookmarks
  ├── resource     (JAX-RS resource classes)
  ├── service      (transactional business logic)
  ├── repository   (PanacheRepository implementations)
  ├── domain       (JPA entities)
  ├── dto          (request and response records)
  └── exception    (custom exceptions and ExceptionMappers)

REST
JAX-RS via Quarkus REST. Resource methods return either DTO records
(serialised to JSON by Jackson) or jakarta.ws.rs.core.Response for
explicit status-code control on create/delete.

ERROR HANDLING
A single package com.workshop.bookmarks.exception contains:
- domain exceptions (NotFoundException, ConflictException, etc.),
- ExceptionMapper implementations for each domain exception,
- one ExceptionMapper for ConstraintViolationException producing RFC 7807
  Problem Details with a 'violations' array describing each field error,
- one fallback ExceptionMapper for Throwable producing a generic 500
  Problem Details (without leaking stack traces).

TESTING
- Unit tests: JUnit 6 + Mockito for service-layer tests.
- Integration tests: @QuarkusTest + RestAssured.
- Dev Services starts PostgreSQL automatically for tests.
- Place integration tests in src/test/java mirroring the production package.

OBSERVABILITY
- quarkus-logging-json produces JSON logs in all profiles.
- quarkus-opentelemetry is enabled with default configuration.
- Resource methods do not log request bodies. Service methods log entity
  IDs and outcomes only.

BUILD
- Gradle wrapper committed to the repository.
- ./gradlew build runs unit and integration tests.
- ./gradlew quarkusDev launches the app in dev mode on port 8080.
- Swagger UI is available at /q/swagger-ui in dev mode.
```

Your agent will produce several planning artifacts:

```
.specify/specs/001-bookmark-manager/
├── spec.md
├── plan.md          ← overall technical plan
├── data-model.md    ← entity and DTO design
├── research.md      ← notes on library and API choices
└── quickstart.md    ← how to run the project
```

## Reviewing the plan

This review is where most of the workshop's real learning happens. Read `plan.md` and `data-model.md` carefully and check for the following:

**Architecture defects to watch for**

- Does the agent put business logic in the resource layer? If yes, push back: resources should be thin and delegate to services.
- Does the data model use a join entity for the bookmark↔tag many-to-many, or a `@ManyToMany` with a generated join table? Either is valid, but the choice should be deliberate. Make the agent justify it.
- Does the agent propose a `BookmarkTag` join entity with its own ID, or use the JPA-managed join table? At this scale either works; the simpler choice is the JPA-managed join table.
- Are timestamps generated by the database (`DEFAULT CURRENT_TIMESTAMP`) or by Java (`Instant.now()` in `@PrePersist`)? Either is fine, but the choice must be consistent across both entities.

**Dependency creep to reject**

The agent may quietly propose any of the following — none of which were authorised:

- Lombok (forbidden by constitution).
- MapStruct or ModelMapper (forbidden).
- `quarkus-resteasy-problem` (not needed; the built-in mechanism is sufficient).
- `quarkus-hibernate-validator-quickstart` or other "quickstart" bundles.
- A REST client extension (not needed; we have no outgoing HTTP calls).
- A security extension (out of scope).

If any of these appear in `plan.md`, instruct the agent to remove them.

**Migration file**

`plan.md` or `data-model.md` should reference `V1__init.sql` and describe (in SQL terms) the tables it creates. If it does not, ask for it explicitly. The migration is the single source of truth for the schema, and Hibernate's `validate` mode will fail at startup if the entities and the migration disagree.

**OpenAPI annotations**

The plan should specify that resource methods carry `@Operation`, `@APIResponse`, and `@Tag` annotations. If it does not, the generated OpenAPI document will be uselessly bare. Push back.

> **⚠️ Do not move on until the plan is clean**
> Once tasks are generated from a flawed plan, fixing the plan means regenerating tasks too. The plan is the cheapest place to catch architectural mistakes.

---

# Part 6 — Generating the Task Breakdown

## Run /speckit-tasks

In your agent's chat interface, type:

```
/speckit-tasks
```

Your agent will generate `tasks.md` at `.specify/specs/001-bookmark-manager/tasks.md`.

A well-generated task list for this project should contain roughly 25–40 tasks, grouped by feature area. Expect something like:

```
## Foundation
- [ ] T001: Configure application.properties (profiles, OTel, JSON logging, Hibernate generation=validate)
- [ ] T002: Create V1__init.sql with bookmark, tag, and bookmark_tag tables
- [ ] T003: Verify the project starts and Flyway applies V1 on first run

## Domain layer
- [ ] T004: Create Bookmark JPA entity
- [ ] T005: Create Tag JPA entity
- [ ] T006: Wire the many-to-many relationship between Bookmark and Tag

## DTO layer
- [ ] T007: Create CreateBookmarkRequest record with Jakarta Validation constraints
- [ ] T008: Create UpdateBookmarkRequest record
- [ ] T009: Create BookmarkResponse record with static fromEntity() factory
- [ ] T010: Create TagResponse record with bookmarkCount field

## Repository layer
- [ ] T011: Create BookmarkRepository (PanacheRepository<Bookmark>) with query methods
- [ ] T012: Create TagRepository (PanacheRepository<Tag>) with findByName, listWithCounts

## Service layer
- [ ] T013: Create BookmarkService with create, list, get, update, delete methods
- [ ] T014: Unit tests for BookmarkService (happy paths and failure modes)
- [ ] T015: Create TagService with list and delete methods
- [ ] T016: Unit tests for TagService

## Exception handling
- [ ] T017: Create NotFoundException, ConflictException, custom exceptions
- [ ] T018: Create ProblemDetail record matching RFC 7807
- [ ] T019: Create ExceptionMapper for each domain exception
- [ ] T020: Create ExceptionMapper for ConstraintViolationException
- [ ] T021: Create fallback ExceptionMapper for Throwable

## Resource layer
- [ ] T022: Create BookmarkResource with all five endpoints + OpenAPI annotations
- [ ] T023: Create TagResource with both endpoints + OpenAPI annotations

## Integration tests
- [ ] T024: BookmarkResource integration tests — happy paths
- [ ] T025: BookmarkResource integration tests — validation and not-found cases
- [ ] T026: TagResource integration tests — happy paths
- [ ] T027: TagResource integration tests — failure cases
- [ ] T028: End-to-end smoke test: create with new tags, list, search, delete

## Verification
- [ ] T029: Confirm Swagger UI renders all endpoints with summaries and responses
- [ ] T030: Confirm logs are JSON and include trace IDs
```

**Review the task list before proceeding.** Cross-check every endpoint and every acceptance criterion in `spec.md` against the task list. Anything missing is going to be missing in the final code.

Look in particular for:

- **A task for each ExceptionMapper.** Easy to omit. Without them, errors return the default Quarkus error page, not RFC 7807.
- **A task creating the Flyway migration.** If `V1__init.sql` is not explicitly a task, the agent may forget it and rely on Hibernate auto-generation — which the constitution forbids.
- **Tests paired with their production code.** No task should produce production code without an accompanying test task (or a single combined task).

> **📝 Tasks are the last checkpoint before code is written**
> Once you run `/speckit-implement`, the agent starts writing files. Course-corrections after that are more expensive. Spend ten minutes here to save an hour later.

---

# Part 7 — Implementation

## Run /speckit-implement

In your agent's chat interface, type:

```
/speckit-implement
```

Your agent will validate that all required artifacts exist (constitution, spec, plan, tasks), then execute the tasks in order. Expect it to:

- Create Java source files in the correct packages.
- Write the `V1__init.sql` Flyway migration.
- Adjust `application.properties` for profiles, logging, OpenTelemetry, and the Hibernate generation strategy.
- Write unit tests (JUnit 6 + Mockito) and integration tests (`@QuarkusTest` + RestAssured).
- Run `./gradlew build` periodically to verify its own work.
- Use Quarkus Dev Services to spin up PostgreSQL when running integration tests.

## What to expect during implementation

This phase takes longer than the beginner workshop — typically 30–60 minutes of agent activity. The agent will:

- Mark each task `[x]` as it completes it.
- Re-run failing tests and attempt to fix them before moving on.
- Ask you when it hits genuine ambiguity not covered by the spec or clarifications. Answer briefly and continue.

> **⚠️ Common agent failures at this stage**
> - **Hibernate validation failure on startup**: the entities and `V1__init.sql` are out of sync. Tell the agent to align them, with the migration as the source of truth.
> - **`@ManyToMany` cascade trap**: deleting a bookmark cascades to delete tags. The spec forbids this. Verify by reading the cascade configuration on `Bookmark.tags`.
> - **Validation errors return 500 instead of 400**: the `ConstraintViolationException` mapper is missing or registered incorrectly. Push back and reference the constitution.
> - **OpenAPI document is empty or missing operation summaries**: resource methods lack `@Operation` annotations. Tell the agent to add them.

## After implementation

Run the full build:

```
./gradlew build
```

All tests should pass. If they do not, do not patch the code yourself — give the failures back to the agent and let it fix them. That feedback loop is part of the workshop.

Then start the app in dev mode:

```
./gradlew quarkusDev
```

Open `http://localhost:8080/q/swagger-ui` in a browser. Every endpoint should be listed with a summary and at least one response schema. Try a `POST /bookmarks` from the Swagger UI:

```json
{
  "url": "https://quarkus.io",
  "title": "Quarkus",
  "description": "Supersonic Subatomic Java",
  "tags": ["java", "framework"]
}
```

You should receive a `201 Created` with a body containing an `id`, the submitted fields, the assigned timestamps, and the two tags. Then:

- `GET /bookmarks` should return your bookmark in a list.
- `GET /tags` should return both tags with `bookmarkCount: 1` each.
- `POST /bookmarks` with `"url": "not-a-url"` should return a `400` with an RFC 7807 body and a `violations` array.
- `GET /bookmarks/99999` should return a `404` with an RFC 7807 body.

If any of these misbehave, find the spec section covering the case and report it back to the agent.

---

# Part 8 — Reflection and Exercises

## What just happened

You produced a non-trivial REST API without writing application code by hand. More importantly, you produced a **specification** that is independent of the implementation. The same `spec.md` could be re-implemented in Spring Boot, in Micronaut, in Go, or in Kotlin, and the behaviour would be identical from the client's perspective.

The discipline you practised is:

1. **Rules first** (constitution). Encode the constraints that should apply project-wide.
2. **WHAT before HOW** (specify and clarify). Describe behaviour without committing to a stack.
3. **HOW in one place** (plan). Tech choices live here and nowhere else.
4. **Tasks before code** (tasks). The last cheap checkpoint.
5. **Implementation is the consequence**, not the starting point.

## Common mistakes to avoid

| Mistake | Why it matters |
|---|---|
| Letting tech leak into `spec.md` | The spec is no longer portable, and the plan no longer has a job. |
| Skipping `/speckit-clarify` at this scale | Guarantees rework. The questions raised here are the questions the agent would have answered silently and probably wrongly. |
| Reading `plan.md` superficially | Architectural mistakes baked into the plan reappear in every subsequent task. |
| Accepting silently added dependencies | Lombok, MapStruct, security extensions — all are forbidden by the constitution. If they appear, your constitution is being ignored. |
| Patching code by hand when tests fail | You lose the feedback loop that improves your spec and your prompts. Give failures back to the agent. |

## Exercises

### Exercise 1 — Orphan tag cleanup (Easy)

The spec states that deleting a bookmark does not delete tags that become orphaned. Add a new specification: "When a bookmark is deleted, any tag that no longer belongs to any bookmark is also deleted." Run `/speckit-specify`, then `/speckit-clarify`, then `/speckit-plan`, `/speckit-tasks`, and `/speckit-implement`. Observe how the agent updates the existing `BookmarkService.delete` method.

### Exercise 2 — PATCH support (Medium)

Currently `PUT /bookmarks/{id}` requires the full resource. Specify a new `PATCH /bookmarks/{id}` endpoint that allows partial updates. Decide carefully: how does PATCH semantics interact with the `tags` field? What does "tags absent in the PATCH body" mean, and how is that distinguishable from "tags present and empty"?

This exercise will force you to confront the difference between "field not provided" and "field provided as null", which is one of the genuinely hard problems in REST API design. Use `/speckit-clarify` aggressively.

### Exercise 3 — Optimistic locking (Medium)

The current spec accepts last-write-wins. Modify the spec to require optimistic locking: each bookmark response includes a `version` field; updates must include the expected `version`; if it does not match the stored version, the API returns `409 Conflict` with a Problem Details body. Walk through all six steps. Pay attention to what changes in the data model and in `V2__add_version.sql`.

### Exercise 4 — Audit the spec (Hard)

Re-read the original `spec.md`. Find at least three behaviours that are **underspecified** — situations the spec does not clearly resolve. Examples to look for: behaviour when the same tag name appears twice in the same request body; behaviour when a search query is an empty string; behaviour when `description` is explicitly `null` vs. absent. Add clarifications and discuss: which of these did your implementation handle correctly anyway, and which did the agent silently guess at?

### Exercise 5 — Stack swap (Hard)

Without touching `spec.md` or the clarifications, write a new `plan.md` targeting Spring Boot 3.x with Spring Data JPA. Run `/speckit-tasks` and `/speckit-implement` in a separate branch. Compare the resulting code with the Quarkus version. Discuss: what survived the stack change unchanged? What changed despite the spec being identical? Did any spec defects become visible only after the second implementation exposed them?

---

# Summary

| Command | What it does |
|---|---|
| `specify init` | Bootstraps the project once. Copies templates and slash commands. |
| `/speckit-constitution` | Sets binding rules for the AI agent across the whole project. |
| `/speckit-specify` | Defines what to build. Technology-agnostic. Produces `spec.md`. |
| `/speckit-clarify` | Resolves ambiguities before planning. Mandatory at this scale. |
| `/speckit-plan` | Introduces the tech stack. Produces `plan.md`, `data-model.md`, `research.md`. |
| `/speckit-tasks` | Breaks the plan into ordered tasks. Produces `tasks.md`. |
| `/speckit-implement` | Executes the tasks. Writes code, tests, configuration, migrations. |

## A note on Panache Next

Quarkus 3.31 introduced **Panache Next**, the next generation of Panache, alongside classic Panache. This workshop deliberately uses **classic Panache** because it is what the agent's training data and the wider community know best. If you repeat this workshop in a year, consider migrating to Panache Next as a stretch exercise.

> **📚 Further reading**
> GitHub Spec Kit repository:   `github.com/github/spec-kit`
> Spec Kit installation guide:  `github.com/github/spec-kit/blob/main/docs/installation.md`
> Full SDD methodology:         `github.com/github/spec-kit/blob/main/spec-driven.md`
> Quarkus documentation:        `quarkus.io/guides`
> Hibernate Panache guide:      `quarkus.io/guides/hibernate-orm-panache`
> RFC 7807 Problem Details:     `datatracker.ietf.org/doc/html/rfc7807`
