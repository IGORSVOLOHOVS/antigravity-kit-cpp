# Antigravity Kit — Agent Architecture

> This repository is powered by a polyglot multi-agent AI system for professional **C++23** and **Python 3.10+** development.

---

## 🗺️ System Map

```
.agent/
├── ARCHITECTURE.md          ← You are here
├── AGENT_FLOW.md            ← Full agent flow diagram & workflow lifecycle
├── mcp_config.json          ← MCP tool configuration
│
├── agents/      (11 files)  ← AI personas, each specializing in a distinct domain
├── rules/       (19 files)  ← Enforced coding & architectural standards
├── skills/      (18 dirs)   ← Domain-specific knowledge modules
├── workflows/   (24 files)  ← Slash command step-by-step procedures
├── prompts/     (7 files)   ← Reusable instruction templates for complex operations
├── scripts/                 ← Automation scripts for linting, testing, and coverage
└── templates/               ← Scaffold templates for new files and components
```

---

## 🤖 Agents (`agents/`)

11 specialist personas. The system auto-selects based on request domain:

| Agent | Specialty |
| :---- | :-------- |
| `cpp-specialist` | C++23 logic, build system, CMake |
| `python-specialist` | Python 3.10+, type safety, packaging |
| `systems-architect` | ISO 25010, C4 diagrams, ADRs |
| `performance-optimizer` | Profiling, nanobench, data-oriented design |
| `security-auditor` | Memory safety, sanitizers, Bandit |
| `debugger` | Systematic crash analysis, ASan/UBSan |
| `qa-automation-engineer` | TDD, doctest, pytest, coverage |
| `documentation-writer` | Doxygen, pdoc, Mermaid diagrams |
| `project-planner` | ISO 25010 planning, task decomposition |
| `orchestrator` | Multi-agent coordination and sub-tasking |
| `code-archaeologist` | Legacy code analysis and migration |

---

## 📏 Rules (`rules/`)

**Binding rule files** — All are enforced automatically by the active agent.

### Base Rules (8 — Required for all projects)
These rules form the foundation of every project and are always included:

| Rule | Purpose |
| :--- | :------- |
| `GEMINI.md` | **P0** — System-wide agent behavior, routing, classification |
| `GIT_WORKFLOW.md` | Semantic commits & branching standards |
| `CPP_ARCHITECTURE.md` | ISO 25010 architecture evaluation |
| `CPP_FUNCTIONAL_CORE.md` | Functional Core, Imperative Shell pattern |
| `CPP_TESTING.md` | 100% doctest coverage, AAA pattern |
| `CPP_CLEAN_ARCH.md` | Hexagonal architecture, zero core dependencies |
| `CPP_TDD.md` | Red-Green-Refactor cycle |
| `CPP_CONCURRENCY.md` | Message passing, Actor model (if multi-threaded) |

### Optional Rules (added by project type)
**If Python included:**
`PYTHON_ARCHITECTURE` · `PYTHON_CLEAN_CODE` · `PYTHON_PACKAGE_MANAGEMENT` · `PYTHON_TESTING`

**If advanced patterns needed:**
`CPP_CQRS` (Command Query Responsibility Segregation) · `CPP_DDD` (Domain-Driven Design) · `CPP_DOD` (Data-Oriented Design)

---

## 🧠 Skills (`skills/`)

Domain-specific knowledge modules loaded on demand by agents.

### Base Skills (8 — Always available)
`architecture` · `cpp-functional-core` · `clean-code` · `testing-and-coverage` · `cmake-build-system` · `memory-management` · `tdd-workflow` · `plan-writing`

### Extended Skills (10 — Loaded as needed)
`architecture-25010` · `bash-linux` · `code-review-checklist` · `docker-ci-cd` · `performance-profiling` · `systematic-debugging` · `vulnerability-scanner` · `pytest-workflow` · `python-static-analysis` · `python-advanced-features`

---

## ⚡ Workflows (`workflows/`)

24 slash-command procedures (12 C++ · 12 Python):

**C++:** `/build-cpp` · `/test-coverage-cpp` · `/benchmark-cpp` · `/refactor-cpp` · `/plan-architecture-cpp` · `/static-analyze-cpp` · `/pack-cpp` · `/docs-cpp` · `/debug-cpp` · `/init-project-cpp` · `/optimize-cpp`

**Python:** `/build-python` · `/format-python` · `/lint-python` · `/test-coverage-python` · `/benchmark-python` · `/refactor-python` · `/plan-architecture-python` · `/static-analyze-python` · `/pack-python` · `/docs-python` · `/debug-python` · `/init-project-python` · `/optimize-python`

---

## 📋 Prompts (`prompts/`)

7 reusable instruction templates for complex operations:

| Prompt | Description |
| :----- | :---------- |
| `git-deploy.md` | Stage → Commit → Push following `GIT_WORKFLOW.md` |
| `verify.md` | Critically audit the previous step against standards |
| `extract-context.md` | Synthesize high-density session summary for context transfer |
| `split-task-by-context.md` | Decompose a large plan into parallel independent sub-tasks |
| `plan-tools-capabilities.md` | Enhance plans with Agents/Rules/Skills checklist |
| `optimize-prompt.md` | Transform rough drafts into expert-level prompts |
| `init-workflow-prompts.md` | Generate a full `workflow_prompts/` library (25 prompts) |

---

## ⚙️ Execution Flow

```
1. ROUTING
   User request → classified in GEMINI.md (P0 rules)
   → auto-selects agent from agents/
   → loads relevant skills from skills/

2. EXECUTION
   Agent pulls rules from rules/
   → follows procedures from workflows/ (or prompts/)
   → applies domain standards (CPP_FUNCTIONAL_CORE / PYTHON_CLEAN_CODE)

3. VERIFICATION
   C++    → python3 .agent/scripts/verify_all.py
              (clang-format, clang-tidy, CMake, doctest, LCOV, nanobench, ASan)

   Python → python3 .agent/scripts/verify_all_python.py
              (ruff format, ruff check, mypy --strict, pytest --cov, bandit)

4. DELIVERY
   Present changes → provide rationale → suggest next steps
```

---

## ❤️ HEARTBEAT.md — Project Health Monitor

The `.agent/HEARTBEAT.md` file tracks real-time project health:

**Auto-generated metrics:**
- Build status (cmake, compiler)
- Test results & coverage
- Code quality score
- Memory safety (ASan/UBSan)
- Dependency status
- Recent issues (last 7 days)

**Generated by:** `python3 .agent/scripts/generate_heartbeat.py`
**Updates:** Automatically in CI/CD or on-demand

This provides a quick health snapshot without diving into detailed logs.

---

## 🚀 Quick Start for New Projects

1. **Copy base files** from this project (`agents/`, `rules/` base set, `skills/` base set)
2. **Run `generate_heartbeat.py`** to establish baseline metrics
3. **Select optional modules** (Python, advanced patterns, DevOps)
4. **Extend as needed** — add specialized rules/skills when project grows

See `.agent/HEARTBEAT.md` for current project status.

---

**Last Updated**: 2026-05-13
**Version**: 2.1.0 (Base + Extended modularity)
