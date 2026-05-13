# Python Package Management & Structure

## 1. The Standard: `uv`
This project utilizes [uv](https://github.com/astral-sh/uv) an extremely fast Python package and project manager. To maintain speed and determinism, it replaces pip, pip-tools, and virtualenv.

- **Initialization**: `uv init`
- **Add Dependency**: `uv add <package>`
- **Add Dev Dependency**: `uv add --dev <package>`
- **Run Scripts**: `uv run <script.py>`
- **Sync Env**: `uv sync`

## 2. pyproject.toml
All project metadata, dependencies, and tool configurations (ruff, mypy, pytest) must reside in a central `pyproject.toml`. Do not use `setup.py`, `setup.cfg`, or `requirements.txt` as the absolute source of truth.

## 3. Project Structure
Use the `src/` layout pattern to prevent accidental imports of the local directory over the installed package.

```text
my_project/
├── pyproject.toml
├── uv.lock
├── src/
│   └── my_package/
│       ├── __init__.py
│       └── core.py
└── tests/
    ├── conftest.py
    └── test_core.py
```
