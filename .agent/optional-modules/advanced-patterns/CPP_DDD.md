---
trigger: model_decision
description: Domain-Driven Design for C++. Enforces Value Objects, Aggregates, Bounded Contexts, and Ubiquitous Language.
---

# CPP_DDD.md — Domain-Driven Design

> **MANDATORY:** Business concepts must never be represented by raw primitives (`int`, `std::string`). Use strongly-typed Value Objects enforced by Concepts.

## 🏷️ Value Objects — Replace Primitives with Types
Use Concepts to statically enforce domain rules at call sites:
```cpp
// Concept: anything that is a valid domain identifier
template<typename T>
concept DomainId = requires(T id) {
    { id.value() } -> std::convertible_to<std::uint64_t>;
    requires std::regular<T>; // equality-comparable and copyable
};

// Concrete Value Objects
struct UserId  { std::uint64_t value() const noexcept; private: std::uint64_t m_v; };
struct OrderId { std::uint64_t value() const noexcept; private: std::uint64_t m_v; };

// Compile error if wrong ID type is passed — no runtime check needed!
template<DomainId Id>
std::expected<User, std::string> find_user(Id id) noexcept;

find_user(UserId{42});  // ✅ compiles
find_user(42);          // ❌ compile error: int does not satisfy DomainId
```

## 📬 Repository Concept — No Virtual Dispatch
Replace `IRepository` pure virtual classes with a **Concept**:
```cpp
// Concept: defines what a UserRepository must support
template<typename Repo>
concept UserRepository = requires(Repo r, UserId id, User u) {
    { r.find_by_id(id) } -> std::same_as<std::expected<User, std::string>>;
    { r.save(u)         } -> std::same_as<std::expected<void, std::string>>;
};

// Use case receives ANY type satisfying the Concept — zero overhead, inlined at compile time
template<UserRepository Repo>
std::expected<void, std::string>
promote_user(Repo& repo, UserId id) noexcept {
    auto user = repo.find_by_id(id);
    if (!user) return std::unexpected(user.error());
    // ... domain logic
    return repo.save(*user);
}

// Both work without virtual, without overhead:
promote_user(sqlite_repo, id);   // SqliteUserRepository
promote_user(mock_repo, id);     // MockUserRepository (for tests)
```

## 🧱 Aggregates — Enforced via Concepts
```cpp
// Concept: anything that can act as an Aggregate Root
template<typename T>
concept AggregateRoot = requires(T agg) {
    { agg.id()          } -> DomainId;
    { agg.take_events() } -> std::same_as<std::vector<DomainEvent>>;
};
```

## 🗺️ Bounded Contexts — Hard Directory Boundaries
```
src/
  billing/      ← Billing Context (invoices, payments)
  inventory/    ← Inventory Context (products, stock)
  shipping/     ← Shipping Context (delivery, addresses)
```
- A `User` in Billing is **not** the same type as a `User` in Shipping.
- Cross-context communication via Domain Events or DTOs — never raw struct sharing.

## 📖 Ubiquitous Language
- ❌ `process_record()` → ✅ `submit_order()`
- ❌ `data_object` → ✅ `CustomerProfile`

## Anti-Patterns
- Using raw `int` or `std::string` as domain identifiers.
- Bypassing the Concept constraint with unconstrained templates.
- Exposing aggregate internals via public getters allowing external mutation.
