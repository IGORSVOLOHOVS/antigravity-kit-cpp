# Optional AI Agent Modules

This directory contains **optional rule modules** that can be added to projects as they grow in complexity or when specific technologies are adopted.

## Structure

### 🐍 Python Rules (`python-rules/`)
Use when your project includes **Python 3.10+** alongside C++.

| File | Purpose |
|------|---------|
| `PYTHON_ARCHITECTURE.md` | Python system design patterns |
| `PYTHON_CLEAN_CODE.md` | Code style, linting, type safety |
| `PYTHON_PACKAGE_MANAGEMENT.md` | Packaging, versioning, distribution |
| `PYTHON_TESTING.md` | pytest, coverage, testing standards |

**How to add:**
```bash
cp -r optional-modules/python-rules .agent/rules/
```

### 🏛️ Advanced Patterns (`advanced-patterns/`)
Use when your C++ project needs **specialized architectural patterns**.

| File | Purpose | When to use |
|------|---------|------------|
| `CPP_DDD.md` | Domain-Driven Design | Complex business logic, bounded contexts |
| `CPP_DOD.md` | Data-Oriented Design | Performance-critical systems, game engines |
| `CPP_CQRS.md` | Command Query Responsibility Segregation | Event-sourced systems, audit trails |

**How to add:**
```bash
cp optional-modules/advanced-patterns/CPP_*.md .agent/rules/
```

### 📚 Practical Guides (`practical-guides/`)
Already covered by **Skills** (`.agent/skills/`), but kept here as reference rules.

| File | Equivalent Skill | Notes |
|------|------------------|-------|
| `CPP_LOGGING.md` | `cpp-functional-core/` | Logging patterns in pure core |
| `CPP_BENCHMARK.md` | `performance-profiling/` | Performance benchmarking |
| `CPP_STATIC_ANALYSIS.md` | `vulnerability-scanner/` | ASan, UBSan, cppcheck |
| `CPP_CI_CD_DOCKER.md` | `docker-ci-cd/` | Multi-stage Docker, CI pipelines |

**Note:** These are implemented as **Skills** in the base system for a cleaner separation:
- **Rules** = "MUST DO" requirements
- **Skills** = "HOW TO" practical guides

---

## Recommended Adoption Path

### Stage 1: New C++ Project (Base)
✅ Start with 8 base rules + 8 base skills
- All C++23 features covered
- Full testing & safety included

### Stage 2: Add Python (+ 4 files)
```bash
cp -r optional-modules/python-rules .agent/rules/
# Add: PYTHON_ARCHITECTURE.md, PYTHON_CLEAN_CODE.md, PYTHON_PACKAGE_MANAGEMENT.md, PYTHON_TESTING.md
```

### Stage 3: Complex Systems (+ 3 files)
```bash
cp optional-modules/advanced-patterns/CPP_*.md .agent/rules/
# Add: CPP_DDD.md, CPP_DOD.md, CPP_CQRS.md
```

### Stage 4: Full Enterprise (+ 1 file)
```bash
cp optional-modules/practical-guides/CPP_CONCURRENCY.md .agent/rules/
# Advanced multi-threading with Message Passing pattern
```

---

## How to Use in New Project

1. **Copy base rules to new project:**
   ```bash
   cp .agent/rules/GEMINI.md <new-project>/.agent/rules/
   cp .agent/rules/GIT_WORKFLOW.md <new-project>/.agent/rules/
   cp .agent/rules/CPP_*.md <new-project>/.agent/rules/
   ```

2. **If you need Python support:**
   ```bash
   cp optional-modules/python-rules/* <new-project>/.agent/rules/
   ```

3. **If you need advanced patterns:**
   ```bash
   cp optional-modules/advanced-patterns/* <new-project>/.agent/rules/
   ```

4. **Update `.agent/ARCHITECTURE.md` to reflect what's included**

---

## Recovery from Git

All files are **permanently stored in git history** and can be recovered:

```bash
# Restore a single file
git checkout HEAD -- .agent/rules/CPP_LOGGING.md

# Restore all deleted files
git checkout HEAD -- .agent/rules/

# Or copy from this archive
cp optional-modules/*/* .agent/rules/
```

---

## File Organization Strategy

**Why separate?**
- **Base rules (8)**: Minimum viable governance for any C++ project
- **Optional rules (11)**: Added complexity, only when needed
- **Skills (18)**: Always available, loaded on demand by agents

This keeps the system **lean by default** and **extensible on demand**.

---

**Last Updated**: 2026-05-13
**Compatibility**: antigravity-kit-cpp v2.1.0+
