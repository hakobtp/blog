# Understanding ACID in PostgresSQL

```info
Author      Ter-Petrosyan Hakob
```

---

In PostgreSQL (and other databases), ACID is a set of properties that ensure each transaction is safe, reliable, and correct. ACID stands for:

- **A**tomicity
- **C**onsistency
- **I**solation
- **D**urability

Let’s look at what each of these means in PostgreSQL.

## Atomicity

**Atomicity** means that a transaction is treated as one complete unit. It either **succeeds entirely** or **fails completely**.

- If one SQL statement in a transaction fails, all the previous changes in that transaction are **automatically rolled back**.
- This ensures that the database is not left in a partial or broken state.

## Consistency

**Consistency** ensures that the database **remains valid** after any transaction. PostgreSQL helps enforce consistency through:

**Constraints**

Rules that ensure the correctness of data:

- **Primary Key** – Every row must be unique and have a clear identity.
- **Foreign Key** – Maintains relationships between tables (e.g., child rows must match parent rows).
- **Unique** – Prevents duplicate values in specific columns.
- **Check** – Validates that values meet a condition (e.g., age > 0).
- **Not Null** – Ensures columns cannot be left empty.

**Triggers**

- These are functions that automatically run when certain actions happen (like **INSERT**, **UPDATE**, or **DELETE**).
- They can be used to apply **business logic** or keep data consistent.
- Example: A trigger that updates a `modified_at` column every time a row is changed.

**Foreign Key Relationships**

- Foreign keys help link tables and ensure **referential integrity**.
- Options like [**ON DELETE CASCADE**](./../cascade/cascade.md) or [**ON UPDATE CASCADE**](./../cascade/cascade.md) make sure that related data stays in sync when a parent row is changed or deleted.


## Isolation

**Isolation** makes sure that transactions running at the same time **don’t interfere** with each other.

- PostgreSQL supports different **isolation levels**:
    - Read Committed
    - Repeatable Read
    - Serializable
- These control how much one transaction can `"see"` the changes from others.
- PostgreSQL uses [MVCC (Multi-Version Concurrency Control)](./3_Multi_Version_Concurrency_Control.md) to allow multiple transactions to read and write data without blocking each other.

## Durability

**Durability** means that once a transaction is committed, its changes are **saved permanently**, even if the system crashes.

- PostgreSQL uses **Write-Ahead Logging (WAL)** to make this happen.
- Changes are first written to a transaction log **before** updating the database itself.
- If the system crashes, PostgreSQL can use the log to recover committed transactions safely.

---

## Explore More

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)
- [Introducing transactions](./1_Introducing_transactions.md)