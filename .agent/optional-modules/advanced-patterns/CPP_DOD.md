---
trigger: model_decision
description: Data-Oriented Design and Entity-Component-System for C++. Maximizes cache locality and SIMD throughput.
---

# CPP_DOD.md — Data-Oriented Design / ECS

> **MANDATORY:** In performance-critical paths, design around the data layout in memory, not around OOP class hierarchies.

## 🧠 Core Principle: Think in Data, Not Objects
Traditional OOP arrays of objects scatter data across memory. DOD reorganizes it for maximum cache line efficiency.

```cpp
// Bad — Array of Structures (AoS): cache-unfriendly for position-only loops
struct Entity {
    float x, y, z;
    float health;
    std::string name;    // 💀 heap allocation per element
};
std::vector<Entity> entities;

// Good — Structure of Arrays (SoA): tight cache locality, SIMD-vectorizable
struct EntityPool {
    std::vector<float> x, y, z;
    std::vector<float> health;
    std::vector<std::string> name;
    std::size_t count = 0;
};
```

## 🎮 ECS with Concept-Constrained Systems
Systems are **concept-constrained pure functions** — no virtual dispatch, zero overhead, fully inlineable:

```cpp
// Concept: anything that looks like a position component store
template<typename T>
concept PositionStore = requires(T s, std::size_t i) {
    { s.x[i] } -> std::convertible_to<float&>;
    { s.y[i] } -> std::convertible_to<float&>;
    { s.z[i] } -> std::convertible_to<float&>;
    { s.count } -> std::convertible_to<std::size_t>;
};

// Concept: anything that looks like a velocity component store
template<typename T>
concept VelocityStore = requires(T s, std::size_t i) {
    { s.dx[i] } -> std::convertible_to<const float&>;
    { s.dy[i] } -> std::convertible_to<const float&>;
    { s.dz[i] } -> std::convertible_to<const float&>;
};

// System — pure function, works on ANY store satisfying the Concepts
template<PositionStore Pos, VelocityStore Vel>
void movement_system(Pos& positions, const Vel& velocities, float dt) noexcept {
    for (std::size_t i = 0; i < positions.count; ++i) {
        positions.x[i] += velocities.dx[i] * dt;
        positions.y[i] += velocities.dy[i] * dt;
        positions.z[i] += velocities.dz[i] * dt;
    }
}
```

The compiler generates **specialized, inlined code** for each concrete store type — equivalent to hand-written assembly, but safe.

## ⚡ Rules for High-Performance Paths
1. **No virtual dispatch in hot loops.** Use Concept-constrained templates for compile-time polymorphism.
2. **No heap allocation inside loops.** Pre-allocate pools or use `std::array`.
3. **Prefer `std::span<T>` over raw pointers** to communicate data slices between systems.
4. **Validate via benchmark, not intuition.** All DOD changes must ship with a `nanobench` report (`CPP_BENCHMARK.md`).

## Anti-Patterns
- Deep `virtual` inheritance trees (`Animal → Mammal → Dog`) that scatter data across the heap.
- Storing `std::string` or any heap-allocated object inside tight data arrays.
- Using `dynamic_cast` in a hot path.
