# good-code-review

A Claude skill that reviews code against the "good code" principles from Catherine Nelson's *Software Engineering for Data Scientists* (O'Reilly, 2024).

The five core principles are **simple, modular, readable, performant and robust**. On top of those it checks error handling and logging, tests, documentation, dependencies, APIs, CI/Docker, security, and ML-specific pitfalls such as data leakage and train/inference skew.

The output is a findings report:
- A scorecard over the principles.
- Each finding ranked 🔴 must fix / 🟡 should fix / 🔵 consider.
- `file:line` and the principle behind every finding.

The skill is review-only: it changes files only when you ask it to.

The five principles apply to any language. Checks marked **(Py)** run only on Python code, and **(ML)** only on machine-learning code.

## Install

### Claude Code: plugin

```
/plugin marketplace add esteban-dept/good-code-review
/plugin install good-code-review@good-code-review
```

To update, run `/plugin marketplace update good-code-review`.

### Claude Code: skills CLI

```
npx skills add esteban-dept/good-code-review
```

### Claude Code: manual

```
git clone https://github.com/esteban-dept/good-code-review.git
cp -R good-code-review/skills/good-code-review ~/.claude/skills/
```

To make it available only inside one project, copy the folder to `<repo>/.claude/skills/` instead.

### claude.ai / Claude desktop

1. Zip the `skills/good-code-review` folder.
2. In claude.ai, go to **Settings → Capabilities** and turn on code execution.
3. Go to **Skills → Upload skill** and choose the zip.

In claude.ai the skill reviews code you paste, upload or connect from GitHub. It can't run your local linters or tests there.

> This repo is private. You need read access to it before any of the install commands above will work.

## Use

| Command | What it reviews |
|---|---|
| `/good-code-review` | Your uncommitted changes |
| `/good-code-review src/pipeline/` | A file or folder |
| `/good-code-review notebooks/eda.ipynb` | A notebook |
| `/good-code-review feature/my-branch` | A branch, compared with the default branch |
| `/good-code-review 123` | A pull request (needs the `gh` CLI) |

You can also ask in plain words, for example "review this code" or "is this production-ready?".

If the project has ruff, mypy, pytest, bandit or pip-audit configured and installed, the skill runs them in check mode first. It never installs anything and never auto-fixes. After the report, ask it to fix specific findings. It applies them one at a time and runs the tests after each.

## Layout

```
skills/good-code-review/
├── SKILL.md                  # steps the review follows, severity levels, report format
└── references/checklist.md   # ~90 checks in sections A–K
.claude-plugin/               # plugin + marketplace manifests
```

To change what the review checks, edit `references/checklist.md`. The severity levels, the 15-finding cap and the report format are in `SKILL.md`.

## Credit

The principles come from *Software Engineering for Data Scientists* by Catherine Nelson (O'Reilly, 2024). This repo contains only a checklist derived from the book, not any of its text. For the reasoning behind the checks, read the book.
