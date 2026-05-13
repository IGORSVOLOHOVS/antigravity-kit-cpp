# CPP_STATIC_ANALYSIS.md - C++ Static Analysis & Sanitizers

> **MANDATORY:** Code must compile cleanly with `-Wall -Wextra -Wpedantic -Werror` and pass Clang-Tidy without warnings.

## 1. Clang-Tidy Mandates
When analyzing the code or generating a `.clang-tidy` config, the following checks are mandatory:
- `bugprone-*`
- `cppcoreguidelines-*` (specifically around memory management and raw pointers)
- `modernize-*` (enforce `auto`, `nullptr`, `override`, `constexpr`)
- `performance-*` (avoid implicit copies `auto` vs `auto&`)

**Exceptions:** We disable exception-related guidelines (`cppcoreguidelines-pro-type-vararg`) in the purely functional `core/` since we exclusively use `std::expected` for monads, not `throw`.

## 2. Sanitizers integration
Any Debug build configuration via CMake MUST inject:
```cmake
add_compile_options(-fsanitize=address,undefined -g)
add_link_options(-fsanitize=address,undefined)
```
- A CI/CD pipeline fails instantly on any ASan (AddressSanitizer) or UBSan (Undefined Behavior) report.

## 3. CppCheck Protocol
If utilizing `cppcheck`, the agent should check for memory leaks and bounds errors:
```bash
cppcheck --enable=all --suppress=missingIncludeSystem --inconclusive --std=c++23 src/
```

## 4. No Circumvention
Never use `#pragma GCC diagnostic ignored` or NOLINT unless it is fundamentally impossible to fix the underlying architecture issue safely.
