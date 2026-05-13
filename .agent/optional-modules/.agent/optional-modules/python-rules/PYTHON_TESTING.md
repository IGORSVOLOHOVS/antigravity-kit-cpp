# Python Testing Standards

## 1. Pytest
- Use `pytest` for all testing. Unittest standard library is discouraged.
- Maintain a separate `tests/` directory at the root of the project.
- Prefix test files with `test_` and postfix test functions with `_test` or prefix with `test_`.

## 2. Fixtures Over Setup/Teardown
- Do NOT use `setup()` or `teardown()` class methods.
- Use pytest `@pytest.fixture` to handle state creation and teardown (using `yield` for teardown).
- Keep fixtures near where they are used, or in `conftest.py` if shared globally at the directory level.

## 3. Mocking & Monkeypatching
- Prefer dependency injection, passing mock objects directly into core functions.
- When unavoidable, use pytest's `monkeypatch` fixture or `unittest.mock` (often via `pytest-mock` plugin).
- Never make network requests or touch local unmanaged files in the test suite unless it's explicitly an integration test.

## 4. Parametrization
- Use `@pytest.mark.parametrize` to test multiple inputs/outputs on the same logic instead of writing repetitive test functions.

## 5. Coverage
- Code coverage is mandatory. Aim for >80% coverage.
- Use `pytest --cov=src` to measure. Code should be tested in CI using coverage gates.
