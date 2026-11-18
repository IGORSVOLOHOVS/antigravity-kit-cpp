# antigravity-kit-cpp: Software Requirements Specification

Reverse-engineered from the code at commit `8555323` on 2026-11-18. It describes what the
repository does now, not what it was meant to do. Overview and known problems:
[docs/README.md](docs/README.md).

## 1. Purpose and scope

The system distributes a set of AI-assistant instruction files (`.agent/`) and editor/CI
templates into other projects. It also provides Python wrapper scripts that the assistant
runs to build, format, lint, test and report on C++23 and Python projects.

In scope:

- The installer and `ai-init` entry point `scripts/install_github.py`.
- The automation scripts `.agent/scripts/*.py` (14 files).
- The release workflow `.github/workflows/publish-rules.yml`.
- The content tree `.agent/`: agents, rules, optional modules, skills, workflows,
  prompts, templates, MCP configuration.

Out of scope:

- The behaviour of the AI editor that interprets `.agent/`. It is not in this repository.
- The MCP server modules named in `.agent/mcp_config.json`. They are not in this repository.
- The prose quality of the instruction files. This spec covers their presence, location
  and machine-read fields, not their advice.
- `.agent/tmp/`. It is shipped (FR-6) but nothing outside it is defined in terms of its
  content.

## 2. Context

Actors:

- Kit user: runs the installer and `ai-init` in a target project. Who the users are is not
  stated (see section 9).
- AI assistant in an editor: reads `.agent/` and runs scripts. `README.md:36` names Cursor
  and Windsurf. The rule frontmatter (`trigger: always_on`, `model_decision`) and the file
  name `GEMINI.md` are consumed by a tool that is not named in the repository.
- Maintainer: pushes to `main`, which triggers the release workflow.

External systems:

- GitHub: clone source (`scripts/install_github.py:11`, `:32`) and releases through the `gh`
  CLI (`.github/workflows/publish-rules.yml:44-56`).
- Command-line tools on `PATH` in the target project: git, cmake, ctest, clang-format,
  clang-tidy, run-clang-tidy, lcov, genhtml, OpenCppCoverage, uv, gcc, clang.

Runtime environment:

- Python 3 with the standard library only (imports in `scripts/install_github.py:2-9` and
  in all `.agent/scripts/*.py`).
- Windows, Linux and macOS (Darwin) for the installer (`scripts/install_github.py:161-169`).
- Windows and non-Windows branches for coverage (`.agent/scripts/run_test_coverage.py:83-86`).
- CI on `ubuntu-latest` (`publish-rules.yml:17`).

## 3. Functional requirements

No requirement below has an automated test; the repository contains none. "Manual" marks
behaviour that was observed in a run on 2026-11-18 (see docs/README.md, "About this
document").

### 3.1 Project initialization (`scripts/install_github.py`)

- FR-1. When run with no flag, the system shall shallow-clone (`--depth 1`) the repository
  at `SOURCE_REPO` into a temporary directory. Source: `scripts/install_github.py:11`,
  `:28-32`, `:172-173`. Verification: no test.
- FR-2. The system shall stop with exit code 1 and print the command and its stderr when a
  git command fails. Source: `scripts/install_github.py:13-19`. Verification: no test.
- FR-3. The system shall stop with exit code 1 when the cloned repository has no `.agent`
  directory. Source: `scripts/install_github.py:37-39`. Verification: no test.
- FR-4. The system shall delete an existing `.agent` in the current directory, printing a
  warning, and replace it with the cloned `.agent`. Source:
  `scripts/install_github.py:41-46`. Verification: no test.
- FR-5. The system shall copy each top-level entry of `.agent/templates/` into the current
  directory only if no entry of that name exists there. Otherwise it shall print a skip
  warning. Source: `scripts/install_github.py:48-60`. Verification: no test.
- FR-6. The system shall copy the whole cloned `.agent` tree, including `tmp/` and
  `templates/`. Source: `scripts/install_github.py:46`. Verification: no test.

### 3.2 Global install and update (`scripts/install_github.py`)

- FR-7. With `--install`, when the process is not elevated, the system shall re-launch
  itself elevated. On Windows it uses `ShellExecuteW` with `runas` and exits 1 if that
  returns 32 or less. Elsewhere it uses `execvp("sudo", ...)`. Source:
  `scripts/install_github.py:64-88`, `:155-157`. Verification: no test.
- FR-8. With `--install` on Windows, the system shall write `ai-init.cmd`, which runs
  `python "<absolute script path>" %*`. It is written into `%SystemRoot%` when elevated,
  otherwise into `%USERPROFILE%\bin`. Source: `scripts/install_github.py:126-147`.
  Verification: no test.
- FR-9. With `--install` on Linux or Darwin, the system shall make the script executable.
  It shall then symlink it as `/usr/local/bin/ai-init` if that directory is writable,
  otherwise as `~/.local/bin/ai-init`. Source: `scripts/install_github.py:98-124`.
  Verification: no test.
- FR-10. With `--install` on any other OS, the system shall exit with code 1. Source:
  `scripts/install_github.py:167-169`. Verification: no test.
- FR-11. With `--update`, the system shall run `git fetch` and then `git pull` in the
  directory two levels above the script file. Source: `scripts/install_github.py:90-96`.
  Verification: no test.
- FR-12. The system shall print usage for `--install` and `--update` on `--help`. Source:
  `scripts/install_github.py:150-153`. Verification: manual (`--help`).

### 3.3 C++ automation scripts (`.agent/scripts/`)

- FR-13. `build.py` shall configure CMake into `build/` with `CMAKE_BUILD_TYPE=Debug` and
  exported compile commands. It shall copy `build/compile_commands.json` to the root if
  present and build with `-j <cpu_count or 2>`. It exits 1 on any failure. Source:
  `.agent/scripts/build.py:15-26`. Verification: no test.
- FR-14. `format.py` shall exit 1 if `clang-format` is not available. Otherwise it runs
  `clang-format -i` on `*.cpp`, `*.hpp`, `*.c`, `*.h` under `src/`, `tests/`, `core/`,
  `shell/` and on `main.cpp`, or reports that no files were found. Source:
  `.agent/scripts/format.py:17-37`. Verification: no test.
- FR-15. `lint.py` shall exit 1 when neither `build/compile_commands.json` nor
  `compile_commands.json` exists. It runs `run-clang-tidy -p build/` when `which` finds it,
  otherwise `clang-tidy` over `*.cpp` in `src/ core/ shell/`. Source:
  `.agent/scripts/lint.py:16-27`. Verification: no test.
- FR-16. `run_static_analysis.py` shall run `run-clang-tidy -p build -j <n> -quiet`. It
  exits 1 when `build/compile_commands.json` is missing, the tool is missing, or the tool
  fails. Source: `.agent/scripts/run_static_analysis.py:11-39`. Verification: no test.
- FR-17. On Windows, `run_test_coverage.py` shall run OpenCppCoverage on every
  `build/**/test_*.exe`, or on `*_test.exe` if there are none. It merges the results into
  HTML in `docs/coverage-report/`, deleting that directory first, and exits 1 if
  OpenCppCoverage or the test executables are missing. Source:
  `.agent/scripts/run_test_coverage.py:21-58`. Verification: no test.
- FR-18. On other systems, `run_test_coverage.py` shall reset lcov counters, run `ctest`
  in `build/`, and capture coverage. It removes `/usr/*`, `*/_deps/*`, `*/tests/*` and
  `*/external/*`, then runs `genhtml` into `docs/coverage-report/`. It exits 1 at the first
  failing step. Source: `.agent/scripts/run_test_coverage.py:60-76`. Verification: no test.
- FR-19. `run_benchmarks.py` shall execute every executable file matching
  `build/**/bench_*` and list `*.html` files in the current directory. Source:
  `.agent/scripts/run_benchmarks.py:11-33`. Verification: no test.
- FR-20. `verify_all.py` shall run `format.py`, `build.py`, `lint.py` and
  `run_test_coverage.py` in that order, stopping with exit 1 at the first failure. Source:
  `.agent/scripts/verify_all.py:13-29`. Verification: no test.

### 3.4 Python automation scripts (`.agent/scripts/`)

- FR-21. `format_python.py` shall run `uv run ruff format .` and exit 1 if it fails or `uv`
  is missing. Source: `.agent/scripts/format_python.py:7-16`. Verification: no test.
- FR-22. `lint_python.py` shall run `ruff check .`, `mypy --strict .`, `vulture .` and
  `bandit -r .` through `uv`. It exits 1 after all four if any failed. It also runs
  `radon cc . -nc` and only prints its findings as a warning. Source:
  `.agent/scripts/lint_python.py:10-66`. Verification: no test.
- FR-23. `run_pytest_coverage.py` shall run
  `uv run pytest --cov=src --cov-report=term-missing --cov-report=html` and exit 1 on
  failure. Source: `.agent/scripts/run_pytest_coverage.py:7-16`. Verification: no test.
- FR-24. `docs_python.py` shall run pdoc on `src` if it exists, otherwise on `.`, with
  output to `docs/`. Source: `.agent/scripts/docs_python.py:10-15`. Verification: no test.
- FR-25. `verify_all_python.py` shall run `format_python.py`, `lint_python.py` and
  `run_pytest_coverage.py` in that order, stopping with exit 1 at the first failure. Source:
  `.agent/scripts/verify_all_python.py:8-26`. Verification: no test.

### 3.5 Commit-message check (`.agent/scripts/check_commit_msg.py`)

- FR-26. The system shall read the commit-message file given as the first argument and
  drop lines starting with `#`. It accepts the first remaining line only if it matches
  `type(scope): description.`, with type in
  `feat|feature|fix|bugfix|docs|refactor|test|chore`. The description is limited to ASCII
  letters, digits, whitespace and `_-.,'"!?`, and must end with `.`. Otherwise it prints
  the rules and exits 1. Source: `.agent/scripts/check_commit_msg.py:6-47`. Verification:
  manual (usage message; both example strings of section 8 step 5 as message files;
  pattern applied to `git log` subjects).
- FR-27. The commit-message template shall wire FR-26 as a `commit-msg` pre-commit hook.
  Source: `.agent/templates/.pre-commit-config.yaml:28-34`. Verification: no test.

### 3.6 Health report (`.agent/scripts/generate_heartbeat.py`)

- FR-28. The system shall write a Markdown report to `--output`, resolved under `--project`
  and creating parent directories. The report has a status summary, a metrics table, a
  dependency list for gcc, clang, cmake and git, up to 5 `git log --oneline` lines (hash and
  subject) for commits of the last 7 days whose message matches fix, bug or issue, and a
  fixed task list. Source:
  `.agent/scripts/generate_heartbeat.py:153-186`, `:188-281`, `:288-303`. Verification:
  manual.
- FR-29. The system shall set the overall status to CRITICAL if any of the build, test or
  memory-safety statuses contains the cross-mark symbol (U+274C), to WARNING if any
  contains the warning-sign sequence (U+26A0 U+FE0F), and to HEALTHY
  otherwise. Source: `.agent/scripts/generate_heartbeat.py:199-203`. Verification: manual
  (WARNING observed).
- FR-30. The system shall report the build as SKIP when `CMakeLists.txt` is absent.
  Otherwise the status reflects the exit code of `cmake --version`. Source:
  `.agent/scripts/generate_heartbeat.py:53-65`. Verification: manual (SKIP observed).
- FR-31. Each probe command shall be killed after 30 s and reported with exit code 124.
  Source: `.agent/scripts/generate_heartbeat.py:36-51`. Verification: no test.

### 3.7 Release (`.github/workflows/publish-rules.yml`)

- FR-32. On a push to `main` that changes `.agent/**`, `README.md` or the workflow file, the
  system shall force-move tag `latest` to the pushed commit and push it. Source:
  `publish-rules.yml:3-10`, `:28-33`. Verification: no test.
- FR-33. The workflow shall then run the health report (FR-28) into `.agent/HEARTBEAT.md`
  on the runner. Source: `publish-rules.yml:35-36`. Verification: no test.
- FR-34. The workflow shall edit release `latest` if it exists, or else create it as
  non-prerelease and latest. In both cases it uploads `scripts/install_github.py`,
  replacing the existing asset. Source: `publish-rules.yml:38-56`. Verification: no test.

### 3.8 Content tree (`.agent/`)

- FR-35. The system shall provide 11 agent files in `.agent/agents/`. Ten of them have a
  `description:` frontmatter. `python-specialist.md` has no frontmatter. Source:
  `.agent/agents/*.md`. Verification: no test (file count).
- FR-36. The system shall provide 8 rule files in `.agent/rules/`. `GEMINI.md` has
  `trigger: always_on`. `CPP_ARCHITECTURE`, `CPP_CLEAN_ARCH`, `CPP_CONCURRENCY`,
  `CPP_FUNCTIONAL_CORE` and `CPP_TDD` have `trigger: model_decision`. `CPP_TESTING` and
  `GIT_WORKFLOW` have no trigger. Source: frontmatter of `.agent/rules/*.md`. Verification:
  no test.
- FR-37. The system shall provide 11 optional rule files outside `.agent/rules/`: 4 in
  `python-rules/`, 3 in `advanced-patterns/` and 4 in `practical-guides/` under
  `.agent/optional-modules/`. An identical nested copy is under
  `.agent/optional-modules/.agent/optional-modules/`, together with its `README.md`.
  Source: `git ls-files`; `diff -r`. Verification: manual (diff).
- FR-38. The system shall provide 24 workflows in `.agent/workflows/`. The 11 `*-cpp.md`
  files use a `description:` frontmatter. The 13 `*-python.md` files use
  `trigger: "/<name>"` and `description:`. Source: frontmatter of `.agent/workflows/*.md`.
  Verification: no test.
- FR-39. The system shall provide 18 skills as `.agent/skills/<name>/SKILL.md` with `name:`
  and `description:` frontmatter, and a catalogue `.agent/skills/doc.md`. Source:
  `.agent/skills/`. Verification: no test.
- FR-40. The system shall provide 7 prompt templates in `.agent/prompts/`. Source:
  `.agent/prompts/`. Verification: no test.
- FR-41. The system shall provide the templates `.editorconfig`, `.pre-commit-config.yaml`,
  `.github/workflows/cpp-ci.yml`, `.github/workflows/python-ci.yml` and the `.vscode/`
  files `settings.json`, `tasks.json`, `launch.json`, `extensions.json`,
  `cpp.code-snippets`, `python.code-snippets` and `keybindings.md`. Source:
  `.agent/templates/`. Verification: no test.
- FR-42. The system shall declare MCP servers `cpp-tools`, `python-lft` and `mcp-cpp`,
  started with `python3 -m cpp_mcp_server`, `python_lft_mcp` and `mcp_cpp`. Source:
  `.agent/mcp_config.json:2-15`. Verification: no test.

### 3.9 Declared but not implemented

| Claim | Source | What the code does |
| --- | --- | --- |
| 19 rules, all in `.agent/rules/` | `README.md:47`, `README.md:159-161` | 8 in `.agent/rules/`, 11 in `.agent/optional-modules/` |
| Workflows split "12 C++ · 12 Python" | `.agent/ARCHITECTURE.md:87` | 11 C++ and 13 Python files |
| `verify_all.py` covers nanobench and ASan | `.agent/ARCHITECTURE.md:125-126`, `.agent/AGENT_FLOW.md` validation layer | Runs format, build, lint and coverage only (`verify_all.py:25-29`) |
| `/test-coverage-python` fails below a coverage threshold | `.agent/AGENT_FLOW.md` (`/test-coverage-python`, step 3) | No `--cov-fail-under` (`run_pytest_coverage.py:9`) |
| `/build-cpp` detects CMake presets | `.agent/AGENT_FLOW.md` (`/build-cpp`, step 1); `README.md:83` | `build.py:15` and `build-cpp.md` use plain `-S . -B build` |
| `/static-analyze-cpp` runs cppcheck | `.agent/workflows/static-analyze-cpp.md:2` | `run_static_analysis.py` runs only `run-clang-tidy` |
| Heartbeat collects nanobench regressions and runs as a pre-commit hook | `generate_heartbeat.py:4`, `:11` | No benchmark probe; no hook entry in `.agent/templates/.pre-commit-config.yaml` |
| Agent frontmatter declares `skills:` | `.agent/rules/GEMINI.md:17` | No agent file has `skills:` |
| Rules `CPP_BENCHMARK.md`, `CPP_STATIC_ANALYSIS.md`, `PYTHON_*.md`, `cpp-test.md` are available | `benchmark-cpp.md:9`, `static-analyze-cpp.md:9`, `python-specialist.md:11`, `qa-automation-engineer.md:24`, `GEMINI.md:74` | Not at the referenced paths. The first three groups are in `optional-modules/`, `cpp-test.md` is in `tmp/rules/` |
| Project at 532 tests, 100% coverage, build OK | `.agent/HEARTBEAT.md:10-27` | No tests or `CMakeLists.txt` in the repository; the generator gives Build SKIP, 0 tests here |
| "What's Cooking" tasks | `generate_heartbeat.py:254-258` | Fixed text, not derived from anything |

## 4. Interfaces

### 4.1 Installer CLI

| Invocation | Effect | Exit codes |
| --- | --- | --- |
| `python3 install_github.py` / `ai-init` | FR-1 to FR-6 in the current directory | 0; 1 on git failure or missing `.agent` |
| `... --install` | FR-7 to FR-10 | 0; 1 on failed elevation or unsupported OS. A failed file write only prints an error (`:123-124`, `:146-147`) |
| `... --update` | FR-11 | 0; 1 on git failure |
| `... --help` | argparse usage | 0 |

### 4.2 Scripts

| Script | Arguments | Reads | Writes | External tools |
| --- | --- | --- | --- | --- |
| `build.py` | none | `CMakeLists.txt` | `build/`, `./compile_commands.json` | cmake |
| `format.py` | none | `src/ tests/ core/ shell/ main.cpp` | sources in place | clang-format |
| `lint.py` | none | `compile_commands.json` | none | run-clang-tidy or clang-tidy, which, find, xargs |
| `run_static_analysis.py` | none | `build/compile_commands.json` | none | run-clang-tidy |
| `run_test_coverage.py` | none | `build/` | `docs/coverage-report/`, `coverage.info`, `coverage_filtered.info`, `coverage_<i>.cov` (deleted afterwards) | lcov, ctest, genhtml / OpenCppCoverage |
| `run_benchmarks.py` | none | `build/**/bench_*` | whatever the binaries write | benchmark binaries |
| `verify_all.py` | none | as its steps | as its steps | as its steps |
| `format_python.py` | none | `.` | sources in place | uv, ruff |
| `lint_python.py` | none | `.` | none | uv, ruff, mypy, vulture, bandit, radon |
| `run_pytest_coverage.py` | none | tests, `src` | `htmlcov/` | uv, pytest, pytest-cov |
| `docs_python.py` | none | `src` or `.` | `docs/` | uv, pdoc |
| `verify_all_python.py` | none | as its steps | as its steps | as its steps |
| `check_commit_msg.py` | `<commit-msg-file>` | that file | none | none |
| `generate_heartbeat.py` | `--project`, `--output` | `CMakeLists.txt`, `**/coverage.json`, `**/.coverage`, `**/asan_output*`, `**/ubsan_output*`, `src/**/*.cpp`, git log | the report file | cmake, clang-tidy, gcc, clang, git |

### 4.3 Slash commands to scripts

| Workflow file | Script it names |
| --- | --- |
| `benchmark-cpp.md:15` | `run_benchmarks.py` |
| `static-analyze-cpp.md:13` | `run_static_analysis.py` |
| `test-coverage-cpp.md:10` | `run_test_coverage.py` |
| `init-project-cpp.md:12` | `format.py`, `lint.py` |
| `format-python.md:12` | `format_python.py` |
| `lint-python.md:12` | `lint_python.py` |
| `test-coverage-python.md:12` | `run_pytest_coverage.py` |
| `docs-python.md:12` | `docs_python.py` |

The other 16 workflows contain instructions only.

### 4.4 Release artifact

| Item | Value | Source |
| --- | --- | --- |
| Tag | `latest` (force-moved) | `publish-rules.yml:32-33` |
| Release title | "Latest Agent Rules" | `publish-rules.yml:46`, `:50` |
| Asset | `install_github.py` | `publish-rules.yml:56` |
| Download URL used in README | `.../releases/latest/download/install_github.py` | `README.md:19` |

## 5. Data

There is no database and no persistent state of the system's own. Files:

| Entity | Format | Location | Producer |
| --- | --- | --- | --- |
| Kit content | Markdown with YAML frontmatter (`trigger`, `description`, `name`, `allowed-tools`) | `.agent/**` | maintainer |
| MCP configuration | JSON `{"mcpServers": {name: {command, args}}}` | `.agent/mcp_config.json` | maintainer |
| Health report | Markdown | `.agent/HEARTBEAT.md` (default) | FR-28 |
| `ai-init` wrapper | `.cmd` batch file / symlink | `%SystemRoot%`, `%USERPROFILE%\bin`, `/usr/local/bin`, `~/.local/bin` | FR-8, FR-9 |
| Build data | `compile_commands.json`, CMake build tree | `build/`, root | FR-13 |
| Coverage | lcov `.info`, OpenCppCoverage binary `.cov`, HTML | root, `docs/coverage-report/`, `htmlcov/` | FR-17, FR-18, FR-23 |
| Python API docs | HTML | `docs/` | FR-24 |

## 6. Non-functional requirements

- NFR-1. The installer and all scripts shall depend only on the Python standard library.
  Source: import lines of `scripts/install_github.py` and `.agent/scripts/*.py`.
- NFR-2. External tool invocations in the health report shall be bounded to 30 s each.
  Source: `.agent/scripts/generate_heartbeat.py:44`.
- NFR-3. Builds and static analysis shall use all reported CPUs. The fallback is 2 for
  builds and 4 for static analysis. Source: `.agent/scripts/build.py:25`,
  `.agent/scripts/run_static_analysis.py:20`.
- NFR-4. Template formatting conventions: UTF-8 and LF everywhere. C++ uses 2-space
  indent and 100 columns. Python uses 4-space indent and 120 columns. `*.bat` and `*.cmd`
  use CRLF. Source: `.agent/templates/.editorconfig:7-55`.
- NFR-5. Template C++ language level shall be C++23. Source:
  `.agent/templates/.vscode/settings.json` (`clangd.fallbackFlags`),
  `.agent/templates/.vscode/tasks.json:177`.
- NFR-6. The release workflow shall run with `contents: write` permission only. Source:
  `publish-rules.yml:12-13`.

## 7. Constraints and dependencies

There are no lock files or package manifests. Versions are pinned only in templates and CI:

| Dependency | Version | Source |
| --- | --- | --- |
| `actions/checkout` | v4 | `publish-rules.yml:20`, template workflows |
| `actions/setup-python` | v5, Python 3.11 (C++ CI), 3.10 (Python CI) | `cpp-ci.yml:19-21`, `python-ci.yml:16-18` |
| `codecov/codecov-action` | v3 | `cpp-ci.yml:42` |
| `astral-sh/ruff-pre-commit` | v0.4.1 | `.pre-commit-config.yaml:3-4` |
| `pre-commit/mirrors-clang-format` | v18.1.1 | `.pre-commit-config.yaml:11-12` |
| `pre-commit/pre-commit-hooks` | v4.6.0 | `.pre-commit-config.yaml:18-19` |
| git, gh | unpinned | `install_github.py:32`; `publish-rules.yml:44-56` |
| cmake, clang tools, lcov, OpenCppCoverage, uv and Python tools | unpinned | section 4.2 |

Further constraints:

- The installer always takes HEAD of the default branch (`scripts/install_github.py:32`).
- `--install` needs administrator rights or sudo (`scripts/install_github.py:155-157`).
- Licence: MIT (`LICENSE`).

## 8. Acceptance

The system meets this spec when:

1. `python3 scripts/install_github.py --help` lists `--install` and `--update` (FR-12).
2. In an empty temporary directory with network access, `python3 <path>/install_github.py`
   produces `.agent/`, `.editorconfig`, `.pre-commit-config.yaml`, `.github/workflows/`
   and `.vscode/`. A second run replaces `.agent/` and skips the four template entries
   (FR-1 to FR-6).
3. File counts match FR-35 to FR-41: `ls .agent/agents | wc -l` gives 11, `.agent/rules` 8,
   `.agent/workflows` 24, `.agent/prompts` 7, and there are 18 `SKILL.md` files.
4. `python3 .agent/scripts/generate_heartbeat.py --output <path outside the repo>` in this
   repository writes a report whose Build line reads SKIP and whose overall status is
   WARNING (FR-28 to FR-30).
5. `check_commit_msg.py` accepts a file containing `feat(api): Add user signin endpoint.`
   and rejects `fix(Laser) resolve null pointer` with exit 1 (FR-26).
6. After a push to `main` touching `.agent/**`, release `latest` exists and carries the
   asset `install_github.py` (FR-32 to FR-34).
7. In a sample CMake project with doctest tests, `python3 .agent/scripts/verify_all.py`
   runs the four steps in order and stops at the first failing one (FR-20). The same check
   with a sample uv project applies to `verify_all_python.py` (FR-25).

Steps 1, 3, 4 and 5 were carried out on 2026-11-18. For step 5, both strings were passed
as message files: the first gave exit 0, the second exit 1. On Windows with redirected
cp1252 output the exit 1 of the rejection comes from a `UnicodeEncodeError` at
`check_commit_msg.py:33`; with `-X utf8` the rules are printed and the script exits 1.
Steps 2, 6 and 7 were not carried out.

## 9. Open questions

1. Who uses the kit: only the owner, or the public? The README addresses a general reader;
   no user group is named.
2. Which editor or agent interprets `trigger: always_on` / `model_decision`,
   `// turbo-all` (`static-analyze-cpp.md:11`) and the `@[...]` references? Inference: the
   name "Antigravity" and `GEMINI.md` point to a Gemini-based editor, but the repository
   does not say so. `README.md:36` names Cursor and Windsurf.
3. What is the minimum Python version for the installer and scripts? None is declared.
   Partly answered by the code: `subprocess.run(..., capture_output=True, text=True)`
   (`scripts/install_github.py:15`, `generate_heartbeat.py:42-43`) needs Python 3.7 or
   later. A scan for newer syntax and methods found none. No run on 3.7 was made.
4. Where do the MCP modules `cpp_mcp_server`, `python_lft_mcp` and `mcp_cpp` come from, and
   how are they installed?
5. How was the committed `.agent/HEARTBEAT.md` (532 tests, 100% coverage) produced, and
   for which project? Partly answered by `git log`: it was added in `ca80061` (2026-05-13),
   the same commit as the current generator. It is not unedited generator output. Its
   "Recent Issues" lists three commits from 2026-04-17 and 2026-04-18, which are more than
   7 days before its timestamp. None of their messages contains fix, bug or issue, and the
   lines carry no hashes, unlike `--oneline` output.
6. Should CI commit or upload the HEARTBEAT it generates (FR-33)? Right now the result is
   discarded.
7. Is `.agent/tmp/` meant to be shipped to every target project (FR-6)?
8. Is the nested `.agent/optional-modules/.agent/optional-modules/` copy intended (FR-37)?
9. Does `releases/latest` resolve to the release of tag `latest`? Tags `v1.0.0` and
   `v2.0.0` also exist (`git tag`); the release list was not checked (no network calls).
   From the code: the workflow creates the release with `--latest=true`
   (`publish-rules.yml:53`), and the `gh release edit` branch (`:45-47`) does not set that
   flag.
10. Which version number is current? `.agent/ARCHITECTURE.md:168` says 2.1.0,
    `.agent/AGENT_FLOW.md:261` says 2.0.0, and the code carries none.
11. Should `check_commit_msg.py` accept the types this repository itself uses (`ci`,
    `config`, `deploy`) and descriptions containing `C++`?
12. Is `--update` expected to work through the `ai-init` wrapper (FR-11), or only from a
    clone?
