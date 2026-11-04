# Good-code review checklist

Items marked **(Py)** apply only to Python, and **(ML)** only to machine-learning code.

---

## A. Core principles: apply to all code

### A1. Simplicity
- [ ] **DRY.** Is any piece of knowledge (a constant, a formula, a column list, a threshold, a path) defined in more than one place? Each should have a single source of truth.
- [ ] Is there dead code, commented-out code, or unused imports, variables or parameters?
- [ ] Is there unnecessary complexity: indirection, abstraction or configurability that nothing uses yet?
- [ ] Could a block be replaced by a built-in or standard-library call (e.g. `sum`, `any`, `collections.Counter`, `pathlib`)?

### A2. Modularity
- [ ] Does each function do one thing, with clearly defined inputs and outputs?
- [ ] Are functions short enough to understand at a glance? Flag anything over about 50 lines or with deep nesting.
- [ ] Do functions depend on global state or hidden side effects instead of parameters and return values?
- [ ] Are I/O (reading files, calling APIs, writing to a DB) separated from pure transformation logic, so the logic can be tested without the I/O?
- [ ] Are configuration values (paths, thresholds, hyperparameters, URLs) hardcoded inside logic instead of passed in or loaded from config?
- [ ] Is a script's top-level code inside `if __name__ == "__main__":` / a `main()` function? **(Py)**

### A3. Readability
- [ ] Do names say what they hold? Flag `df2`, `tmp`, `data_new`, `x1`, and single letters outside short loops and math. Is each name the right length?
- [ ] Python naming: `snake_case` for functions and variables, `CamelCase` for classes, `ALL_CAPS` for constants. **(Py)**
- [ ] Do any names shadow built-ins (`list`, `dict`, `id`, `type`, `input`, `sum`)? **(Py)**
- [ ] Are there magic numbers or strings with no named constant or explanation?
- [ ] Is formatting consistent? This should be enforced by a tool (Black or Ruff), not by reviewers.

### A4. Performance
*Raise these only when data size or a requirement makes them matter.*
- [ ] Is a loop nested inside another over the same collection (O(n²)) where a dict or set lookup would do?
- [ ] Is `x in some_list` used inside a loop where a `set` would be O(1)? **(Py)**
- [ ] pandas: are `iterrows()` / `itertuples()` / Python loops used where a vectorized operation exists? `apply` is a middle ground. **(Py)**
- [ ] NumPy: Python loops over arrays instead of vectorized ops; unnecessarily large dtypes (`float64` where `float32` or `int8` would do) on big arrays. **(Py)**
- [ ] Is a whole dataset loaded into memory when it might not fit? Consider chunking, Dask or Polars. **(Py)**
- [ ] Is expensive work recomputed (e.g. inside a loop) when it could be cached or hoisted?
- [ ] Is there a performance claim with no measurement behind it? Ask for a `timeit` or profiler result.

### A5. Robustness
- [ ] **Reproducibility:** are random seeds set where results depend on them? Are input data versions and paths explicit? **(ML)**
- [ ] Are unexpected inputs (empty data, missing columns, NaN, wrong types, zero rows after filtering) handled or explicitly rejected?
- [ ] Does the code assume things about its data (column order, sort order, uniqueness) without checking them?

---

## B. Errors, logging, debugging
- [ ] Is there a bare `except:` or `except Exception:` that swallows errors, or one that only `pass`es / `print`s? **(Py)**
- [ ] Are exceptions caught at the narrowest scope, for the specific exception types expected?
- [ ] Do raised errors say what went wrong **and** what value caused it? (`raise ValueError(f"expected positive n_rows, got {n_rows}")`)
- [ ] Are custom exception classes used where callers need to tell failure modes apart?
- [ ] Do errors fail fast at the boundary (validate inputs early) rather than deep inside?
- [ ] Are `print` statements used for operational output in production or long-running code? It should use `logging`. **(Py)**
- [ ] Are logging levels used sensibly (DEBUG / INFO / WARNING / ERROR)? Is the logger configured once at the entry point rather than in library modules?
- [ ] Do logs leak secrets or personal data?
- [ ] Are there leftover debugging artefacts: `breakpoint()`, `pdb`, stray `print`, `# TODO remove`?

---

## C. Formatting, linting, type checking
- [ ] Is a formatter configured (Black or Ruff format)?
- [ ] Is a linter configured (Ruff, Flake8, Pylint)?
- [ ] Are type hints present on public functions? Is a type checker (mypy, pyright) configured? **(Py)**
- [ ] Do the type hints tell the truth (e.g. `-> pd.DataFrame` on a function that can return `None`)?
- [ ] Do these run automatically, through pre-commit or CI?

---

## D. Testing
- [ ] Does new or changed logic come with tests? This is the most important item in this section.
- [ ] Do the tests follow **arrange / act / assert** (plus cleanup), with one behaviour per test?
- [ ] Are edge cases covered: empty input, single row, NaN, boundary values, wrong types?
- [ ] Do the tests use a framework (pytest), not manual runs or notebook cells? **(Py)**
- [ ] Are fixtures and parametrization used instead of copy-pasted tests?
- [ ] Do tests depend on network, external services, the current date, or execution order? They should be isolated.
- [ ] **Data validation:** are input DataFrames checked against a schema (Pandera, Pydantic, Great Expectations), or at least by asserting column names and types? **(Py)**
- [ ] **(ML)** Are there tests for the parts that *are* deterministic: preprocessing, feature engineering, output shape and range, the model training on a tiny dataset without error, a performance floor on a fixed eval set?

---

## E. Design and refactoring
- [ ] Does the project have a standard, recognisable layout (`src/`, `tests/`, `data/`, `notebooks/`, config)?
- [ ] Are interfaces (function signatures, module boundaries) designed around purpose, not around how the code happened to evolve?
- [ ] **Notebooks:** is production logic stuck in a notebook? Reusable pieces should become functions in a module that the notebook imports.
- [ ] Is a refactor mixed with a behaviour change in the same diff? Ideally they are separate, and tests pass before and after.

---

## F. Documentation
- [ ] Do public functions and classes have docstrings giving purpose, inputs (with types and units), outputs and raised errors?
- [ ] Do comments add information the code can't (why, caveats, sources, units), rather than repeating what it does? (`# train the model` above `model.fit(...)` is noise.)
- [ ] Are there stale comments or docstrings that contradict the code?
- [ ] Does the README say what the project does, how to install it, how to run it, and how to run the tests?
- [ ] **Notebooks:** descriptive filename, a title and purpose at the top, Markdown cells explaining each step, executed top to bottom in order (no out-of-order execution counts), outputs cleared or meaningful.
- [ ] **(ML)** Are experiments tracked in a structured way (MLflow, W&B, or at least a results log), not just in notebook outputs?

---

## G. Version control, dependencies, packaging
- [ ] Are dependencies declared (`pyproject.toml`, `requirements.txt`) **with versions pinned or bounded**? Is there a lock file for applications? **(Py)**
- [ ] Are imports used in the code but missing from the dependency file, or vice versa?
- [ ] Are data files, model artefacts, `.env` files, credentials or large binaries committed? Check `.gitignore`.
- [ ] Does each commit or PR do one coherent thing, with a message explaining *why*?
- [ ] For a package: valid `pyproject.toml` metadata, an importable structure, and no `sys.path` hacks.

---

## H. APIs
- [ ] Are request and response bodies validated with schemas (e.g. Pydantic models in FastAPI)?
- [ ] Are status codes correct (4xx for client errors, 5xx for server errors), never `200` with an error body?
- [ ] When *calling* APIs: is `response.status_code` / `raise_for_status()` checked? Are timeouts set? Are retries and backoff used where appropriate?
- [ ] Are model loading and other heavy initialisation done once at startup, not per request?
- [ ] Are endpoints and parameters named clearly and consistently?

---

## I. Automation and deployment
- [ ] Are there pre-commit hooks for formatting, linting and secret scanning?
- [ ] Does CI run the tests (and lint) on every push or PR?
- [ ] Dockerfile: pinned base image, dependencies installed before code is copied (layer caching), not running as root, no secrets baked in.
- [ ] Is environment-specific config (dev / prod) separated from code?

---

## J. Security
- [ ] Are there **secrets in code or in committed files** (API keys, tokens, passwords, connection strings)? They should come from environment variables or a secret manager. 🔴
- [ ] Is sensitive or personal data committed, logged, or written to unprotected locations? 🔴
- [ ] Is `pickle` / `joblib` / `torch.load` used on files that could come from an untrusted source? Prefer safe formats (safetensors, ONNX, JSON) or verify provenance. **(Py)**
- [ ] Is `eval`, `exec`, `subprocess(..., shell=True)` or string-built SQL used with user-controlled input?
- [ ] Are third-party packages legitimate and current? Watch for typosquatting, abandoned packages and known CVEs (run `pip-audit`).
- [ ] APIs: authentication on non-public endpoints, input size limits, no internal error details in responses.
- [ ] Is a static analysis tool (Bandit) run automatically? **(Py)**
- [ ] **(ML)** Is training data validated before training (poisoning)? Is the deployed model monitored for drift and abuse? Could a new model be redeployed quickly if a problem is found?

---

## K. ML-specific correctness **(ML)**
*These are the main source of silent wrong results in DS code.*
- [ ] **Data leakage:** is the scaler, encoder, imputer or feature selector fit on the full data before the train/test split? Are target-derived features present? Does a time-series split shuffle future data into training?
- [ ] Are seeds set for the split, the model, and the framework (NumPy, PyTorch, TF)?
- [ ] Is the evaluation metric appropriate for the problem (e.g. accuracy on an imbalanced target)?
- [ ] Is the same preprocessing applied at training time and inference time, ideally by the same code path or pipeline object?
