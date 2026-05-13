# CPP_LOGGING.md - Structured Logging Standards

> **MANDATORY:** Pure functions inside the `core/` directory MUST NOT log anything to console, file, or network.

## 1. Architectural Boundaries
- **Core (Functional):** Returning `std::expected` is the ONLY way errors are propagated. Logging is a side-effect. Pure logic does not have side effects.
- **Shell (Imperative):** The `shell/` is responsible for unpacking `std::expected` and logging the failure.

## 2. Structured Logging
Use a high-performance structured logging library (e.g., `spdlog` or `fmt::format` wrappers).
- Never use generic `std::cout << ... << std::endl;` for application-level logs.
- Use log levels aggressively: `TRACE`, `DEBUG`, `INFO`, `WARN`, `ERROR`, `CRITICAL`.

### Bad Example (Core Level Logging):
```cpp
// BAD: Core function doing I/O
std::expected<int, std::string> calculate() {
    spdlog::info("Calculating...");
    return 42;
}
```

### Good Example (Shell Level Logging):
```cpp
// GOOD: Core returns data, Shell logs it
auto result = calculate();
if (!result) {
    spdlog::error("Calculation failed: {}", result.error());
} else {
    spdlog::info("Result: {}", *result);
}
```

## 4. Error Reporting & Stacktraces
- **Stacktraces**: For Debug builds, programs SHOULD utilize C++23 `<stacktrace>` or `backward-cpp` to provide meaningful context on crashes.
- **Sanitizers**: AddressSanitizer (ASan) and UndefinedBehaviorSanitizer (UBSan) MUST be enabled in the CMake `Debug` configuration for "always-on" error detection.
- **Clang Diagnostics**: For projects using Clang, the following flags SHOULD be enabled in CMake to ensure rich error reporting:
    - `-fcolor-diagnostics`: Mandatory visual highlighting.
    - `-fdiagnostics-show-template-tree`: Tree-view comparison for template mismatches.
    - `-fspell-checking`: AI-like "did you mean...?" suggestions for typos.
    - `-fdiagnostics-show-option`: Shows the name of the flag controlling the error.
    - `-fdiagnostics-column-number`: Mandatory for precise location highlighting.
    - `-fdiagnostics-show-note-include-stack`: Better visibility of error origination.
