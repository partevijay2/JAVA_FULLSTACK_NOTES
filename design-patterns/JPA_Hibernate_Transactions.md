# JPA, Hibernate and Transactions — Practical Guide

This note covers JPA fundamentals, Hibernate specifics, transaction semantics, locking strategies, common pitfalls, and typical use-cases. It includes examples and recommendations for Spring Boot + Spring Data JPA usage.

Contents
- Big picture: JPA vs Hibernate
- Entities & mappings (relations, cascades, fetch strategy)
- Persistence context and lifecycle
- JPQL and Criteria
- Transactions: propagation, isolation, rollback rules
- Locking: optimistic, pessimistic, StaleObject, LockModeType
- Common problems: LazyInitializationException, N+1 selects, batching, ID strategies
- Hibernate-specific features and useful objects/methods
- Examples: optimistic locking retry, pessimistic lock example, transaction propagation demo, N+1 problem and fetch-join fix
- Exercises and checklist

Big picture
- JPA: specification (EntityManager, JPQL, annotations). Use it for portability and clarity.
- Hibernate: popular JPA provider with additional features (second-level cache, batch fetching, multi-table mappings). Many production apps use Hibernate as the JPA implementation.

Entities & mappings
- @Entity, @Table, @Id, @GeneratedValue
- Relationships: @ManyToOne, @OneToMany, @OneToOne, @ManyToMany. Prefer LAZY loading to avoid unexpected joins.
- Owner side (the side with the foreign key) must be updated to persist association changes.
- Cascades: use sparingly. CascadeType.PERSIST and MERGE are common for parent-child where lifecycle is bound.
- Orphan removal removes child rows when removed from parent collection.

Persistence context
- EntityManager (JPA) / Session (Hibernate) maintain a first-level cache for the lifetime of transaction/persistence context.
- A managed entity is tracked; changes are flushed to DB at commit or when em.flush() is called.
- detach() removes an entity from persistence context; merge() reattaches a detached instance.

JPQL & Criteria
- JPQL queries operate on entity names and properties: "SELECT e FROM Employee e WHERE e.department.name = :d"
- Criteria API offers type-safe programmatic queries (useful for dynamic queries).

Transactions
- A transaction groups DB operations to commit or rollback.
- Spring @Transactional settings:
  - propagation: REQUIRED (default), REQUIRES_NEW, NESTED, SUPPORTS, NOT_SUPPORTED, MANDATORY, NEVER
  - isolation: DEFAULT, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE, READ_UNCOMMITTED (DB-dependent)
  - rollbackFor / noRollbackFor to customize exception-driven rollback behavior
- In Spring, annotate service-layer methods (public) with @Transactional. Do not rely on controller-level transactions.
- Beware: calling an @Transactional method from the same class (this.someMethod()) bypasses proxy and will not create a transaction.

Locking strategies
- Optimistic locking: @Version field. Lightweight; detects concurrent modifications at commit via version check. Preferred when conflicts are rare.
- Pessimistic locking: LockModeType.PESSIMISTIC_WRITE / PESSIMISTIC_READ to acquire DB-level locks (SELECT ... FOR UPDATE). Use when conflicts are frequent or strong consistency required.
- Handling optimistic failures: catch ObjectOptimisticLockingFailureException (or OptimisticLockException) and retry with backoff if appropriate.

Common pitfalls
- LazyInitializationException: occurs when lazy association is accessed outside a persistence context (e.g., after transaction closed). Fixes: fetch join, DTO projection, open-in-view (controversial), initialize before detach.
- N+1 selects: happens when fetching parent then iterating children lazily. Fix: JOIN FETCH, @BatchSize, or DTO query.
- IDENTITY generation and batching: IDENTITY disables JDBC batching. Use SEQUENCE (or TABLE with batch-friendly strategies) for high-throughput inserts.

Hibernate-specific objects & methods (quick reference)
- SessionFactory (Hibernate) / EntityManagerFactory (JPA): factory for sessions / EntityManagers.
- Session / EntityManager: unit of work and first-level cache.
- Transaction (Hibernate) / EntityTransaction (JPA): begin(), commit(), rollback().
- Query / TypedQuery: createQuery, setParameter, getResultList(), getSingleResult().
- CriteriaBuilder / CriteriaQuery: programmatic query builder.
- Hibernate.initialize(object): force initialization of a proxy or collection.
- Session#save(), persist(), merge(), update(), saveOrUpdate() — Hibernate-native vs JPA-standard differences (prefer JPA where possible).
- StatelessSession: bypasses first-level cache for high-throughput bulk operations (no dirty checking).
- Second-level cache: configured at SessionFactory level (ehcache, infinispan).

Examples (Spring Boot + Spring Data JPA sample)
- The sample project demonstrates:
  - optimistic locking (Employee @Version, concurrent update retry)
  - pessimistic locking (select for update via repository or EntityManager)
  - transaction propagation differences (REQUIRED vs REQUIRES_NEW)
  - N+1 problem and fix using fetch join

Files in example project (design-patterns/examples/jpa-hibernate-sample):
- pom.xml
- src/main/java/com/example/jpa/Application.java
- src/main/java/com/example/jpa/entity/Department.java
- src/main/java/com/example/jpa/entity/Employee.java
- src/main/java/com/example/jpa/repository/EmployeeRepository.java
- src/main/java/com/example/jpa/repository/DepartmentRepository.java
- src/main/java/com/example/jpa/service/EmployeeService.java
- src/main/java/com/example/jpa/service/TransactionalDemoService.java
- src/main/java/com/example/jpa/Runner.java (CommandLineRunner to populate and demo behaviors)
- application.yml

Exercises
1. Implement optimistic locking retry logic on a balance transfer and measure conflicts under concurrency.
2. Convert an operation to use pessimistic lock and compare throughput and failed transactions under contention.
3. Detect and fix an N+1 problem by writing a test that counts queries (use datasource proxy / log inspectors) and apply JOIN FETCH.

Checklist for production
- Model associations with LAZY by default and use explicit joins or DTOs for read models.
- Keep transactions short and at service boundary.
- Prefer optimistic locking; use pessimistic only when necessary.
- Use SEQUENCE ids if you need batching for inserts.
- Monitor SQL counts and slow queries; fix N+1 issues proactively.

---

Created by GitHub Copilot assistant — JPA, Hibernate, and transaction notes with examples.
