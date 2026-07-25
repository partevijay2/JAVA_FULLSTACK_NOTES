# Database

## 1. Normalization (Data Integrity)

The process of organizing tables to eliminate data redundancy and prevent update anomalies.

* **1NF (Atomic Values):** One cell = One value. No arrays or comma-separated lists in a single column.
* **2NF (Whole Key Rule):** Must be 1NF. No partial dependencies. If you have a composite primary key, every non-key column must depend on the *entire* combined key, not just half of it.
* **3NF (Nothing But the Key Rule):** Must be 2NF. No transitive dependencies. Non-key columns cannot depend on other non-key columns (e.g., `Department_Name` should not be in the `Employees` table; it belongs in a `Departments` table).

## 2. Denormalization & NoSQL (Read Speed)

The deliberate act of adding redundant data back into a database.

* **The Goal:** Eliminate slow, CPU-heavy `JOIN` operations.
* **The Trade-off:** Lightning-fast read speeds (`SELECT`), but slower, more complex writes (because you must update duplicated data in multiple places).
* **NoSQL Connection:** Document databases (like MongoDB) are denormalized by default. They store flexible, nested JSON-like objects (similar to the DTOs you pass in a Spring application), completely avoiding the need for relational joins.

## 3. Handling Massive Data: Partitioning vs. Sharding

| Strategy | Concept | Server Location | Best For |
| --- | --- | --- | --- |
| **Partitioning** | Slicing one massive table into smaller sub-tables. | Same physical server | **Range** (Dates), **List** (Regions), **Hash** (Math formula for even distribution). |
| **Sharding** | Breaking the database itself into separate pieces. | Multiple different servers | Infinite horizontal scaling and high availability. Uses a "Shard Key" to route traffic. |

* **Interview Buzzword:** **Partition Pruning**. This is when the database's Query Optimizer looks at your `WHERE` clause and instantly ignores (prunes) all partitions that don't contain your data, saving massive amounts of memory.

## 4. Enterprise Architecture: Event Sourcing & CQRS

* **Event Sourcing (The Ledger):** Instead of storing the *current state* of an object (which overwrites old data), you store an append-only list of immutable events (e.g., `Created`, `Updated`, `Deleted`). Provides a perfect audit trail and allows "time travel" to past states.
* **CQRS (Command Query Responsibility Segregation):** Splitting your data architecture into two separate pathways:
* **Commands:** Write operations that change state (executed on the Event Store).
* **Queries:** Read operations that only fetch data (executed on a separate, heavily denormalized read-database).



## 5. Indexing & Optimization (The Speed Hacks)

Indexes are separate data structures (usually B-Trees) that prevent the database from doing a **Full Table Scan** (reading every single row on the hard drive).

* **Clustered Index:** Dictates the *actual physical order* of the data on the disk. (You can only have **1** per table; it is almost always the Primary Key).
* **Non-Clustered Index:** A separate lookup list with pointers to the data's location. (You can have **multiple** per table).
* **Composite Index Trap (Leftmost Prefix Rule):** If you index `(Last_Name, First_Name)`, the database organizes it like a phone book. A query searching *only* by `First_Name` cannot use the index and will trigger a Full Table Scan. You must query from left to right.
* **The Write Penalty:** Do not index every column. Every time you `INSERT` or `UPDATE`, the database must recalculate the indexes, slowing down write performance.

## 6. The EXPLAIN Command

Prepending `EXPLAIN` to a SQL query shows the Query Optimizer's execution plan. Look for these three columns:

1. **`type`:** You want `ref` or `const` (Index scan). You do *not* want `ALL` (Full Table Scan).
2. **`key`:** Shows the actual name of the index the database chose to use.
3. **`rows`:** The estimated number of rows the database has to scan. The smaller, the better.
