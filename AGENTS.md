# AGENTS.md -- DOSBoxLauncher

Guidelines for AI coding agents working in this repository.
DOSBoxLauncher is a Python-based launcher/frontend for DOSBox.
Licensed under MIT (Copyright 2026 Linwei).

---

## Build & Run

```bash
# Install dependencies (once pyproject.toml is set up)
pip install -e ".[dev]"

# Run the application
python -m dosboxlauncher
```

If a `pyproject.toml` or `requirements.txt` exists, always install deps before
running or testing. Prefer editable installs (`pip install -e .`) for development.

---

## Testing

```bash
# Run entire test suite
pytest

# Run a single test file
pytest tests/test_foo.py

# Run a single test function
pytest tests/test_foo.py::test_bar

# Run a single test class method
pytest tests/test_foo.py::TestBaz::test_qux

# Run tests matching a keyword expression
pytest -k "keyword"

# Run with verbose output
pytest -v

# Run with coverage
pytest --cov=dosboxlauncher --cov-report=term-missing
```

Tests live in the `tests/` directory. Mirror the source tree structure:
`dosboxlauncher/config.py` -> `tests/test_config.py`.

---

## Linting & Formatting

```bash
# Lint (Ruff)
ruff check .

# Lint and auto-fix
ruff check --fix .

# Format (Ruff)
ruff format .

# Check formatting without modifying
ruff format --check .

# Type checking (mypy)
mypy dosboxlauncher/
```

Always run `ruff check` and `ruff format` before committing. Fix all lint
errors; do not add `noqa` comments unless there is a documented reason.

---

## Code Style

### General

- **Python version**: Target Python 3.10+ (use modern syntax: `X | Y` unions,
  `match` statements, etc.).
- **Line length**: 88 characters max (Black/Ruff default).
- **Indentation**: 4 spaces. No tabs.
- **Quotes**: Double quotes for strings (`"hello"`). Single quotes are
  acceptable only inside f-strings or to avoid escaping.
- **Trailing commas**: Always use trailing commas in multi-line collections,
  function signatures, and function calls.
- **Docstrings**: Use Google-style docstrings for all public modules, classes,
  and functions.

### Imports

- Group imports in this order, separated by blank lines:
  1. Standard library (`os`, `sys`, `pathlib`, ...)
  2. Third-party packages (`PyQt5`, `dosbox`, ...)
  3. Local/project imports (`from dosboxlauncher import ...`)
- Use absolute imports. Avoid relative imports except within a package's
  internal modules.
- Never use wildcard imports (`from module import *`).
- Sort imports alphabetically within each group. Let Ruff (`isort` rules)
  handle this automatically.

### Naming Conventions

| Element            | Convention       | Example                  |
|--------------------|------------------|--------------------------|
| Modules/packages   | snake_case       | `config_loader.py`       |
| Functions          | snake_case       | `load_config()`          |
| Variables          | snake_case       | `game_path`              |
| Constants          | UPPER_SNAKE_CASE | `DEFAULT_DOSBOX_PATH`    |
| Classes            | PascalCase       | `GameProfile`            |
| Private members    | `_leading_under` | `_parse_entry()`         |
| Type variables     | PascalCase       | `T`, `ConfigT`          |

### Type Annotations

- Add type annotations to all function signatures (parameters and return type).
- Use `from __future__ import annotations` at the top of every module for
  PEP 604 style unions and forward references.
- Prefer built-in generics (`list[str]`, `dict[str, int]`) over
  `typing.List`, `typing.Dict`.
- Use `None` return annotation explicitly: `def foo() -> None:`.
- Use `TypeAlias` for complex types: `GameList: TypeAlias = list[GameProfile]`.

### Error Handling

- Catch specific exceptions; never use bare `except:` or `except Exception:`.
- Use custom exception classes for domain errors, inheriting from a common
  project base exception.
- Include context in error messages: `raise FileNotFoundError(f"Config not found: {path}")`.
- Use `logging` module, not `print()`, for diagnostic output.
- Let unexpected exceptions propagate; do not silently swallow errors.

### File & Path Handling

- Use `pathlib.Path` for all filesystem operations.
- Never hardcode path separators; let `pathlib` handle OS differences.
- Use `Path.resolve()` to normalize paths before comparison.

---

## Project Structure (Intended)

```
DOSBoxLauncher/
  dosboxlauncher/         # Main package
    __init__.py
    __main__.py           # Entry point (python -m dosboxlauncher)
    config.py             # Configuration loading/saving
    launcher.py           # DOSBox process management
    profiles.py           # Game profile models
    ui/                   # UI layer (if applicable)
  tests/                  # Test suite
    conftest.py           # Shared fixtures
    test_config.py
    test_launcher.py
  docs/
    concept.md            # Project concept/brief
    guide.md              # Project guidelines
    prd.md                # Project requirements
    design.md             # Technical design document
    spec.md               # Technical specification
  pyproject.toml          # Project metadata, deps, tool config
  AGENTS.md
  LICENSE
  README.md
```

---

## Commit & PR Conventions

- Write short imperative commit messages: `"add game profile export feature"`.
- Keep commits focused -- one logical change per commit.
- Branch from `master`. Use `feature/`, `fix/`, `refactor/` prefixes for
  branch names.

---

## Dependencies

- Pin direct dependencies with minimum versions in `pyproject.toml`.
- Dev dependencies (pytest, ruff, mypy) go in `[project.optional-dependencies]`
  under a `dev` extra.
- Do not commit `.env` files or secrets. The `.gitignore` already excludes them.

---

## Key Reminders for Agents

1. Always read existing code before modifying it.
2. Run `ruff check . && ruff format --check . && mypy dosboxlauncher/` before
   finishing work. Fix any issues found.
3. Run `pytest` after any code change and ensure all tests pass.
4. Do not introduce new dependencies without justification.
5. Keep the public API surface small; prefer private helpers.
6. When in doubt, match the style of surrounding code.
