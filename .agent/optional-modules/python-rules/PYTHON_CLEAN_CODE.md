# Python Clean Code & Standards

## 1. Ruff is the Law
All Python formatting and linting MUST adhere to the defaults provided by `ruff`. We do not debate formatting; `ruff format` dictates it.

## 2. Strict Type Hinting
- Every function signature **must** have type hints for its arguments and return type.
- Python 3.10+ style annotations are mandatory (`dict[str, int]` instead of `Dict[str, int]`, `str | None` instead of `Optional[str]`).
- Code must pass `mypy --strict`.
- Avoid `Any` wherever possible. Use `TypeVar` or generics when needed.

## 3. Docstrings
- Use Google or NumPy docstring format for public modules, classes, and exported functions.
- Private internal functions can omit full docstrings if the type hints and name make the behavior self-evident, but a single line description is encouraged.

## 4. Modern Python Idioms
- Favor list comprehensions and generator expressions over map/filter.
- Use `pathlib.Path` instead of `os.path`.
- Use f-strings for string interpolation.
- Prefer `dataclasses` (or Pydantic models if doing validation) instead of boilerplate `__init__` methods.
- Use `match`/`case` pattern matching (Python 3.10+) for complex branching.

## 5. Naming
- **Variables / Functions**: `snake_case`
- **Classes**: `PascalCase` (or `CamelCase`)
- **Constants**: `UPPER_SNAKE_CASE`
- **Private Variables / Methods**: Prefix with `_` (e.g., `_my_private_method`).

## 6. Code Complexity (Radon)
- Keep cyclomatic complexity low. Functions should ideally score an **A** or **B** (Complexity <= 10).
- Highly complex components should be refactored into smaller, cohesive blocks.
- The `lint_python.py` script checks complexity via `radon` and will emit warnings for complex code.

## 7. Security (Bandit)
- The code must be free of known security vulnerabilities (SQL injections, hardcoded credentials, weak cryptography).
- The `lint_python.py` script automatically utilizes `bandit` to scan for high-severity vulnerabilities.

## 8. Dead Code (Vulture)
- Ensure all functions, classes, and variables are utilized.
- Checked using `vulture` during static analysis.

## 9. Pre-commit Hooks
- Commit formatting is enforced using `.pre-commit-config.yaml`. It's highly recommended to run `uv run --with pre-commit pre-commit install` to enable it.

## 10. Observability & Debugging
- **Rich Tracebacks**: All entry points (CLI, Main scripts) SHOULD install `rich.traceback` for beautiful, localized error reporting.
```python
from rich.traceback import install
install(show_locals=True)
```
