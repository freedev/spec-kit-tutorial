## Repo structure

For a single-workshop repo that's designed to accept more workshops later, the standard pattern is `workshops/<slug>/` with shared infrastructure at the root. Here's what I'd recommend:

```
spec-kit-workshops/                   ← repo root
├── README.md                          ← landing page; lists all workshops
├── LICENSE                            ← pick one; CC-BY-4.0 for prose, MIT for code samples
├── CONTRIBUTING.md                    ← how to propose / add a new workshop
├── CODE_OF_CONDUCT.md                 ← Contributor Covenant 2.1, standard
├── .github/
│   ├── ISSUE_TEMPLATE/
│   │   ├── workshop-bug.md            ← "step X doesn't work"
│   │   └── workshop-proposal.md       ← "I want to contribute a new one"
│   ├── PULL_REQUEST_TEMPLATE.md
│   └── workflows/
│       └── markdown-lint.yml          ← lint all .md files on PR
├── workshops/
│   └── number-guessing-game-java/     ← one folder per workshop, kebab-case slug
│       ├── README.md                  ← the workshop itself (your current file, cleaned up)
│       ├── prerequisites.md           ← extracted, so it can be linked from anywhere
│       ├── assets/                    ← screenshots, diagrams if any
│       ├── reference-solution/        ← the finished project, for instructors
│       │   ├── .specify/
│       │   ├── app/
│       │   └── ...
│       └── troubleshooting.md         ← FAQ of things that go wrong
├── docs/
│   ├── what-is-spec-kit.md            ← shared explainer, linked by every workshop
│   ├── when-to-use-sdd.md             ← benefits AND counterindications (the missing piece)
│   └── glossary.md                    ← constitution, spec, plan, tasks definitions
└── templates/
    └── new-workshop-template/         ← skeleton for contributors
        ├── README.md                  ← with section headings filled in
        └── prerequisites.md
```

**Why this shape, specifically:**

- `workshops/<slug>/` rather than top-level folders or branches — keeps the root clean, makes adding a workshop a single-PR operation, and lets GitHub render `workshops/foo/README.md` directly when someone navigates there.
- `docs/` for cross-workshop content (what is Spec Kit, when to use it, glossary) — currently you'd have to repeat this in every workshop. Pull it out once and link to it. This is also where your "benefits and counterindications" section lives, since it applies to all workshops, not just the guessing game.
- `reference-solution/` matters more than people expect. If a learner gets stuck three steps in, they need somewhere to compare against. You'll also want it for testing your own workshop hasn't drifted.
- `templates/new-workshop-template/` is the lever that makes "leave space for others" actually work — contributors copy this folder, rename it, and fill it in. Without a template, every contribution will have a different shape and you'll spend the rest of your life normalizing them.
- I'd avoid putting the workshop content directly at the root README. The root README should be a landing page that points to workshops; the workshop README is the content. Conflating them makes adding workshop #2 awkward.

A naming note: `number-guessing-game-java` rather than `number-guessing-game` — when someone contributes the same exercise in Python or Go (which they will), the slug needs the language suffix to disambiguate.

---

## Root README structure

Keep the landing page short. Its job is orientation, not teaching. Roughly:

1. **One-paragraph "what is this repo"** — a collection of hands-on workshops for learning Spec-Driven Development with GitHub Spec Kit.
2. **"What is Spec-Driven Development?"** — three or four sentences, then a link to `docs/what-is-spec-kit.md` for depth.
3. **"Should I use SDD?"** — two or three sentences acknowledging it isn't always the right tool, then a link to `docs/when-to-use-sdd.md`. This is what makes the repo feel honest rather than evangelical, which matters for community reception.
4. **Available workshops** — a table with columns: Workshop, Language/Stack, Difficulty, Duration, Status. One row per workshop. Easy to extend.
5. **Prerequisites at a glance** — note that each workshop has its own, link to them.
6. **How to contribute a new workshop** — one paragraph, then link to `CONTRIBUTING.md`.
7. **License** — short note, link to `LICENSE`.

The workshop table is the key extensibility primitive. When you add workshop #2, you add a row. That's it.

---

## Presenting it to the community

A few things that materially affect reception:

- **Don't oversell.** The Spec Kit ecosystem already has tutorials that overclaim ("AI writes your code for you, just write a spec!"). Your workshop is honest about review checkpoints and the agent's failure modes — keep that tone in the announcement.
- **Lead with the differentiator.** Most Spec Kit tutorials use TypeScript/Node toy apps. A Java + Gradle workshop is a relatively underserved niche. Say so.
- **Where to announce:** the GitHub Spec Kit Discussions tab (they have one and it's active), Reddit r/java for the Java angle, and Hacker News only if you have a non-tutorial angle ("what I learned trying to teach SDD to Java developers" plays better than "I made a tutorial"). Avoid LinkedIn-style hype phrasing on the dev-facing channels — it reads as marketing and gets ignored.
- **License the prose under CC-BY-4.0 and the code under MIT or Apache-2.0.** Mixing them in a tutorial repo is normal and signals you've thought about reuse.

Want me to draft the actual root `README.md`, the `CONTRIBUTING.md`, and a `when-to-use-sdd.md` (covering benefits and counterindications honestly)? Those are the three files that determine how the repo lands. I'd rather write them once you've decided whether to fix the Java-25-vs-Gradle-9 issue, since the workshop table needs to state the actually-tested versions.