# Design Patterns Index

This directory contains concise notes and examples for common design patterns and architectural topics.

Available notes:

- [Singleton](Singleton.md) — creational pattern ensuring a single instance.
- [Factory Method](FactoryMethod.md) — creational pattern that defers instantiation to subclasses.
- [Microservices](Microservices.md) — essential microservices design patterns, resiliency, integration, and operational patterns.
- [Backend for Frontend (BFF)](BackendForFrontend.md) — client-specific APIs, aggregation, security, ownership, and trade-offs.
- [Locking](Locking.md) — Java locking and concurrency patterns.
- [JPA / Hibernate / Transactions](JPA_Hibernate_Transactions.md) — ORM, transactions, isolation levels, locking, and common pitfalls.
- [ThreadLocal](ThreadLocal.md) — ThreadLocal best practices and examples.

Examples:

- examples/resilience4j-springboot — small Spring Boot sample demonstrating Circuit Breaker and Retry (Resilience4j).
- examples/jpa-hibernate-sample — Spring Boot sample demonstrating optimistic/pessimistic locking, transaction propagation, and N+1 fix (planned).

If you want additional patterns or example projects, tell me which pattern to add next and whether to include runnable code or tests.
