# Contributing

> 🌐 **English** · [Português](./CONTRIBUTING.pt-BR.md)

Thanks for helping improve this collection of learning projects! This guide
explains how the repository is organized and how to contribute well.

## The one rule that matters most

Every project brief is a **guided exercise, not a finished solution.** Describe
*what* to build and *why* it matters; sketch data shapes and contracts — but
never paste a full, copy-paste implementation. The learner writes the code. PRs
that hand over complete solutions will be asked to trim them back.

## Repository structure

```text
projects/
  <domain>/                 backend, frontend, data-science,
    <level>/                data-engineering, devops, system-design
      NN-<slug>/            beginner | intermediate | advanced
        README.md           project brief in English
        README.pt-BR.md     same brief in Portuguese
      README.md             track landing page (links every project)
```

Each domain × level holds 10 numbered projects.

## Ways to contribute

| I want to… | How |
|---|---|
| Suggest a new project | Open a **💡 New project idea** issue first, then a PR. |
| Fix a link, error, or typo | Open a PR (or a **📝 Content issue**). |
| Improve/expand an existing brief | Open a PR against that project's `README.md` **and** `README.pt-BR.md`. |
| Translate a brief | Add/patch the missing-language file so both stay in sync. |
| Improve tooling/docs | Open an **✨ Enhancement** issue or a PR. |

## Adding a new project

1. Fork and create a feature branch.
2. Pick the correct `domain/level` and the next free number.
3. Create the folder `NN-<kebab-case-slug>/`.
4. Copy [`.github/PROJECT_TEMPLATE.md`](./.github/PROJECT_TEMPLATE.md) →
   `README.md` and [`.github/PROJECT_TEMPLATE.pt-BR.md`](./.github/PROJECT_TEMPLATE.pt-BR.md)
   → `README.pt-BR.md`, then fill both in.
5. Add a table row linking the project in the track's `README.md`.
6. Open a PR.

### Project brief structure

Every brief follows the template and includes: **Overview, Prerequisites,
Learning Objectives, Functional Requirements, Suggested Milestones, Data &
Interface Sketch, Stretch Goals, Definition of Done, Common Pitfalls, Resources.**

### Difficulty criteria

- **Beginner** (hours–days): basic knowledge, few dependencies, one core concept.
- **Intermediate** (days–weeks): APIs/DBs/frameworks, several features together.
- **Advanced** (weeks+): architecture, scalability, security, integrated systems.

## Pull request guidelines

- Keep each PR focused on a **single theme**.
- Make sure every relative link you add resolves (CI checks this).
- Keep bilingual files in sync — update **both** languages, or note why not.
- Write clear commit messages (see below).
- Fill in the PR template checklist.

### Commit message format

Use [Conventional Commits](https://www.conventionalcommits.org/):

```text
feat(backend): add rate-limited API project to intermediate
fix(frontend): correct broken link in kanban board brief
docs: expand learning objectives for URL shortener
ci: add markdown lint workflow
```

## Code of conduct

Participation is governed by our [Code of Conduct](./CODE_OF_CONDUCT.md).

## License

By contributing, you agree your contributions are licensed under the repository's
[MIT License](./LICENSE).

---

Thank you for helping make this resource better for everyone!
