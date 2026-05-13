---
trigger: model_decision
description: Rules for C++ CI/CD, CMake setup, Packaging via CPack, and Docker multi-stage builds.
---

# C++ CI/CD, CMake & Docker Rules

## 1. CMake Configuration
- In `CMakeLists.txt`, always use the modern target-based paradigm (`target_include_directories`, `target_link_libraries` with `PUBLIC`/`PRIVATE`).
- Global `include_directories()` are strictly forbidden.
- Use `FetchContent` to manage dependencies seamlessly via source (e.g., `doctest`, `fmt`, `spdlog`).

## 2. Docker & Multi-stage Builds
- Enforce multi-stage Docker builds to keep images minimal:
  - **Stage 1 (Builder):** Base image with GCC/Clang/MSVC, install `cmake`, `ninja`. Compile the project and execute test suites via `ctest`.
  - **Stage 2 (Runtime):** A minimal base image (e.g., Alpine or Ubuntu-slim) bringing in only the final binaries and their dynamic `.so` / `.dll` dependencies.

## 3. Packaging
- Configure CPack within CMake to generate `.deb`, `.rpm`, and `zip` archives for software distribution.
