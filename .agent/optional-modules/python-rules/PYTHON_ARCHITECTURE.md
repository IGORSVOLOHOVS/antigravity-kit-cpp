---
trigger: model_decision
description: Architectural standards enforcing ISO/IEC 25010:2023, Quality Attribute Scenarios, C4 Diagramming, and ADRs for Python.
---

# Python Architecture Standards

Python components must be designed for maintainability and scalability.

## 1. Clean Architecture (Hexagonal)
- Keep business logic isolated from external frameworks and IO (APIs, databases, UI).
- Define "Ports" as Python `Protocol`s (from `typing`) to define interfaces that adapters will implement.
- This creates an observable, testable boundary around your core logic.

## 2. Dependency Inversion
- Depend on abstractions (Protocols / Abstract Base Classes), not concrete implementations.
- Inject dependencies rather than instantiate them inside business layers.
- For complex projects, consider utilizing dependency injection patterns via factory methods.

## 3. SOLID Principles
- **S**ingle Responsibility: A class should have one reason to change.
- **O**pen/Closed: Open for extension, closed for modification.
- **L**iskov Substitution: Subclasses must be substitutable for their base classes.
- **I**nterface Segregation: Many client-specific protocols are better than one general-purpose interface.
- **D**ependency Inversion: High-level modules should not depend on low-level modules; both should depend on abstractions.

## 4. Concurrency vs Asyncio
- Use `asyncio` for I/O bound work (network requests, DB queries).
- Use `multiprocessing` or `concurrent.futures.ProcessPoolExecutor` for CPU-bound work (matrix math, heavy processing) to bypass the GIL.
- Do not mix sync and async indiscriminately; keep the async boundaries clear.
