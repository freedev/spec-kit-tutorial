<p align="center">
  <img src="media/logo_large.webp" alt="Spec Kit Logo" width="200" height="200">

</p>

# Spec Kit Workshops

A collection of hands-on workshops for learning Spec-Driven Development with [GitHub Spec Kit](https://github.com/github/spec-kit).

See `docs/what-is-spec-kit.md` for an overview of Spec-Driven Development.

See `docs/when-to-use-sdd.md` for when SDD is (and isn't) the right tool.

## Available Workshops

| Workshop                   | Language/Stack | Difficulty | Duration | AI Tool | Status  |
|---------------------------|---------------|------------|----------|---------|---------|
| [Number Guessing Game (Java)](workshops/number-guessing-game-java/README.md) | Java/Gradle    | Beginner   | 1 hour   | Claude | Beta    |
| [Number Guessing Game (Python)](workshops/number-guessing-game-python/README.md) | Python/uv      | Beginner   | 1 hour   | Claude | Beta    |
| [Number Guessing Game (Quarkus/Gemini)](workshops/number-guessing-game-quarkus-gemini/README.md) | Quarkus/Java/Maven | Beginner   | 1 hour   | Gemini/Copilot | Beta    |
| [Bookmark Manager REST API (Java)](workshops/bookmark-manager-rest-java/README.md) | Quarkus/Java/Gradle | Intermediate | 3–4 hours | Claude | Beta    |

Each workshop has its own prerequisites (see their folders).

Want to contribute? See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

Prose: [CC-BY-4.0](LICENSE)
Code: [MIT](LICENSE)

---

## Adapting Workshops to Other AI Assistants

While each workshop targets a specific AI agent (like Claude Code or GitHub Copilot), you can easily adapt any workshop to your preferred AI assistant by adjusting the Spec Kit command notation.

**⚠️ Command naming differs by AI agent:**
- **Claude Code** uses hyphen notation: `/speckit-constitution`, `/speckit-specify`, `/speckit-plan`, etc.
- **GitHub Copilot / Google Gemini** use dot notation: `/speckit.constitution`, `/speckit.specify`, `/speckit.plan`, etc.

To use a workshop with a different assistant, simply swap the notation in the instructions for the one your agent supports.
