---
trigger: model_decision
description: CQRS pattern for C++. Strict separation between write (Command) and read (Query) models via Concepts.
---

# CPP_CQRS.md — Command Query Responsibility Segregation

> **MANDATORY:** A function either *changes state* (Command) or *returns data* (Query) — never both. Handlers are constrained via Concepts, not virtual dispatch.

## 🧠 Core Principle: CQS at the Function Level
```cpp
// ✅ Query — [[nodiscard]], returns data, changes nothing
[[nodiscard]] std::expected<UserProfile, std::string>
get_user_profile(UserId id) const noexcept;

// ✅ Command — returns only success/failure signal, no data
std::expected<void, std::string>
update_user_email(UserId id, Email new_email) noexcept;

// ❌ VIOLATION — mutates AND returns business data
User& get_or_create_user(UserId id);
```

## 📦 Command & Query Concepts
```cpp
// Concept: a valid Command — has an execute() returning void-expected
template<typename T>
concept Command = requires(T cmd) {
    { cmd.execute() } -> std::same_as<std::expected<void, std::string>>;
};

// Concept: a valid Query — has a fetch() returning some non-void expected
template<typename T>
concept Query = requires(T q) {
    typename T::ResultType;
    { q.fetch() } -> std::same_as<std::expected<typename T::ResultType, std::string>>;
};

// Generic command bus — works for any type satisfying Command
template<Command Cmd>
std::expected<void, std::string> dispatch(Cmd&& cmd) noexcept {
    return cmd.execute();
}

// Generic query bus — works for any type satisfying Query
template<Query Qry>
auto dispatch(Qry&& q) noexcept {
    return q.fetch();
}
```

## ✍️ Write Side — Command Aggregates
```cpp
// Write Model: enforces domain invariants, emits domain events
class UserAggregate {
public:
    static std::expected<UserAggregate, std::string>
        create(Email email, Age age) noexcept;

    std::expected<void, std::string>
        change_email(Email new_email) noexcept;

    std::vector<DomainEvent> take_events() noexcept;

private:
    UserId m_id;
    Email  m_email;
    Age    m_age;
};

// Concrete command satisfying the Command Concept
template<UserRepositoryConcept Repo>
struct UpdateEmailCommand {
    Repo&   repo;
    UserId  user_id;
    Email   new_email;

    std::expected<void, std::string> execute() noexcept {
        auto user = repo.find_by_id(user_id);
        if (!user) return std::unexpected(user.error());
        user->change_email(new_email);
        return repo.save(*user);
    }
};
static_assert(Command<UpdateEmailCommand<MockUserRepository>>);
```

## 📖 Read Side — Query View Models
```cpp
// Read Model — flat, denormalized, optimized for rendering
struct UserProfileView {
    UserId      id;
    std::string display_name;
    std::string email;
    std::string formatted_age;
};

// Concept: a valid view store for user profiles
template<typename T>
concept UserViewStore = requires(T s, UserId id) {
    { s.get(id) } -> std::same_as<std::expected<UserProfileView, std::string>>;
};

// Concrete query satisfying the Query Concept
template<UserViewStore Store>
struct GetUserProfileQuery {
    using ResultType = UserProfileView;
    Store& store;
    UserId id;

    std::expected<ResultType, std::string> fetch() const noexcept {
        return store.get(id);
    }
};
static_assert(Query<GetUserProfileQuery<MockUserViewStore>>);
```

## Rules
1. **`[[nodiscard]]` on every Query.** If you return data, the caller must use it.
2. **Commands return `std::expected<void, Error>`.** No business data flows out of a mutation.
3. **Read models are separate structs** — never return the domain Aggregate to the UI layer.
4. **No `virtual` dispatch in handlers.** The `Command` and `Query` Concepts replace vtables.
5. **`static_assert` every concrete handler** against its Concept before shipping.

## Anti-Patterns
- A method `save_and_get()` that persists to DB and returns the saved entity simultaneously.
- Using the same `User` domain Aggregate as a DTO returned to the API layer.
- Skipping `static_assert` — without it, compile errors appear at instantiation sites far from the source.
