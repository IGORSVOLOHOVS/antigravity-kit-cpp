# CPP_BENCHMARK.md - Performance Benchmarking Standards

> **MANDATORY:** All C++ performance benchmarks must strictly utilize the `ankerl::nanobench` library. No `google-benchmark` or custom `std::chrono` timers unless explicitly requested as a fallback.

## 1. Library & Setup
- Always specify `#define ANKERL_NANOBENCH_IMPLEMENT` exactly *once* in the benchmark executable.
- Include `<nanobench.h>`.

## 2. Benchmark Structure
All benchmarks must target specific "Hot Path" logic in the *Functional Core*.
```cpp
#include <nanobench.h>

void run_benchmark() {
    ankerl::nanobench::Bench bench;
    bench.title("Benchmark: Specific Computation")
         .unit("ops")
         .warmup(100)
         .relative(true);
         
    bench.run("Scenario 1", [&] {
        auto result = complex_math_function();
        ankerl::nanobench::doNotOptimizeAway(result);
    });
}
```

## 3. Mandatory Reporting
- **Never** just print to stdout for automated pipelines.
- You must generate an HTML report for every benchmark run using the built-in templating.
```cpp
#include <fstream>
std::ofstream file("benchmark_report.html");
bench.render(ankerl::nanobench::Templates::htmlBoxplot(), file);
```

## 4. Anti-Patterns
- Forgetting `doNotOptimizeAway`, resulting in the compiler optimizing the entire loop to zero.
- Profiling network or shell I/O. Benchmarks should focus strictly on the deterministic `core/` algorithms.
