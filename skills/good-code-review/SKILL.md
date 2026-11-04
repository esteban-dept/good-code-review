---
name: good-code-review
description: Review code against the "good code" principles from Catherine Nelson's *Software Engineering for Data Scientists* — simplicity, modularity, readability, performance, robustness — plus testing, error handling and logging, documentation, dependencies, APIs, security and ML-specific risks. Produces a severity-ranked findings report with file:line references and the principle behind each finding. Use when the user asks to "review this code", "review my PR/branch/notebook", "is this production-ready", "SE4DS review", "good-code review", or wants a data-science / ML codebase checked before sharing or deploying it. Review-only by default; fixes only on request.
---

# Good-code review

Review code the way the book defines good code: **simple, modular, readable, performant, robust** (Ch. 1), backed up by tests, documentation, sound dependencies and security. The book is written for Python data science, so its Python- and ML-specific rules apply only when the code is Python / ML. The five core principles apply to any language.

The detailed checklist is in `references/checklist.md`. Read it before reviewing. Each item there carries the book chapter it comes from.

## 1. Fix the scope

Work out what to review from the user's words, in this order:

| User gives | Review |
|---|---|
| A PR number or URL | `gh pr diff <n>` plus enough surrounding code to judge it |
| A branch name | `git diff <default-branch>...<branch>` |
| A path (file, dir, notebook) | That path, whole |
| Nothing, with uncommitted changes | `git diff HEAD` (staged and unstaged) |
| Nothing, clean tree | Ask: whole repo, or a specific area? Never review a whole large repo unprompted. |

Skip vendored and generated paths (`.venv/`, `venv/`, `node_modules/`, `dist/`, `build/`, `.git/`, `__pycache__/`, `.claude/worktrees/`, lock files) and anything the project's CLAUDE.md says to exclude.

## 2. Classify the code

Write down which of these apply. Each one switches on extra checklist sections:

- **Notebook** (`.ipynb`): notebook hygiene and the notebook-to-script path (Ch. 8, 9)
- **Script / pipeline**: modularity, config, logging, reproducibility
- **Library / package**: interfaces, docstrings, packaging, typing (Ch. 6, 9, 10)
- **API service** (FastAPI, Flask, etc.): API design and API security (Ch. 11, 13)
- **ML training / inference**: ML testing, data validation, model security (Ch. 7, 13)
- **Data-heavy** (pandas, NumPy, Polars, Dask): data structures and vectorization (Ch. 2, 3)

## 3. Let the tools go first (read-only)

The book's rule (Ch. 6) is to automate formatting, linting and type checking, not do them by hand. So:

1. Look for the project's own configuration: `pyproject.toml`, `setup.cfg`, `.pre-commit-config.yaml`, `ruff.toml`, `mypy.ini`, CI workflows.
2. If a tool is configured **and already installed**, run it in check mode over the scope only: `ruff check`, `ruff format --check` / `black --check`, `mypy`, `pytest -q`, `bandit -r`, `pip-audit`. Never run a tool's auto-fix mode, never install anything, and never run commands that touch the network or external services unless the user agrees.
3. Report tool output as **one summary line per tool** (count and worst examples). Don't restate every lint warning as a finding of your own.
4. If no formatter, linter or test runner is configured, that absence is itself a finding (Ch. 6, 12).

## 4. Review against the checklist

Go through `references/checklist.md` section by section for the categories that apply. Spend your attention on what tools **can't** catch: design, naming, error-handling choices, test quality, reproducibility, data leakage, security of data and secrets.

For every candidate finding:

- **Verify it.** Read the surrounding code and the callers. Drop anything you can't point at concretely. A shorter report of real findings beats a long speculative one.
- **Be specific.** Give `file:line`, what is wrong, a concrete consequence (what input or situation breaks it, or what it costs), and a suggested fix (a short code sketch where it helps).
- **Respect the project's conventions.** If the team already has a standard (formatter, naming, structure), judge against that, not against the book's defaults (Ch. 6: "comply with your team or company's standards").
- **Don't optimize what doesn't need it.** Raise performance only when data size or a stated requirement makes it matter (Ch. 2: "If the amount of data is small … don't spend time optimizing it").

## 5. Rank by severity

| Severity | Meaning | Typical examples |
|---|---|---|
| 🔴 **Must fix** | Wrong results, a crash, a security or data exposure, or results that can't be reproduced | bug, secret in code, bare `except` hiding failures, train/test leakage, unpinned non-reproducible pipeline, `pickle.load` on untrusted input |
| 🟡 **Should fix** | Works today but is fragile, slow at expected scale, or hard to change | missing tests for new logic, no input validation, `iterrows` over large frames, 200-line function, duplicated logic |
| 🔵 **Consider** | Readability and maintainability polish | naming, comments that repeat the code, missing docstring on an internal helper |

## 6. Report

Use this structure:

```markdown
## Good-code review: <scope>

**Context:** <notebook | script | package | API | ML …> · <n files, n lines reviewed>
**Tools:** ruff: 12 issues (mostly E501) · mypy: not configured · pytest: 34 passed

### Scorecard
| Principle | Rating | One-line reason |
|---|---|---|
| Simplicity | ✅ / ⚠️ / ❌ | … |
| Modularity | … | … |
| Readability | … | … |
| Performance | … | … |
| Robustness | … | … |
| Tests | … | … |
| Documentation | … | … |
| Security | … | … |
(Leave out rows that don't apply. Add ML or API rows when they do.)

### Findings
#### 🔴 Must fix
1. **<short title>** — `path/file.py:42` · *Robustness (Ch. 5)*
   <what is wrong, and the concrete consequence>
   **Fix:** <suggestion, with a code sketch if useful>

#### 🟡 Should fix
…

#### 🔵 Consider
…

### What's working well
<2–4 bullets. Name specific good practice, so it's kept.>

### Suggested next steps
<Ordered, at most 5. Prefer adding automation (pre-commit, CI) over one-off fixes when the same issue recurs.>
```

Cap the report at **15 findings** unless the user asks for all. If you drop some, say how many and in which categories.

## 7. After the report

- Don't edit files unless the user asks. When they do, fix one finding at a time, smallest change first, and run the tests after each (Ch. 8: refactoring workflow).
- Offer to save the report as Markdown if the project keeps review notes.
