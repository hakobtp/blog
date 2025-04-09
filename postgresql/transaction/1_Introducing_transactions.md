# Introducing transactions

```info
Author      Ter-Petrosyan Hakob
```

---

**Transactions** let a database group several operations together and treat them as one single unit, called an **atomic operation**. 
This means that either **all the changes happen**, or **none at all**—ensuring consistency.

PostgreSQL has a powerful and standards-compliant transaction system. It allows users to define specific **transaction properties**, including support for **nested transactions** using savepoints.

PostgreSQL relies on transactions to keep data **consistent and safe**, even when many users or processes are working at the same time. It uses a system called **Write-Ahead Logging (WAL)** to protect data and make recovery possible if something goes wrong.

To support high performance with many simultaneous transactions, PostgreSQL also uses [Multi-Version Concurrency Control (MVCC)](./3_Multi_Version_Concurrency_Control.md). This allows multiple transactions to run at the same time without blocking each other.

A **transaction** is a group of actions that are treated as **one single unit**. It either completes fully or fails completely—there is no in-between.

Transactions are a core feature of any modern database system. They help databases follow the four important [ACID](./0_Understanding_ACID_in_PostgreSQL.md) properties:

- **Atomicity** – All steps in the transaction are done or none at all.
- **Consistency** – The database always moves from one valid state to another.
- **Isolation** – Transactions run as if they were the only ones happening.
- **Durability** – Once a transaction is committed, the changes are saved permanently.

Together, these properties make sure that data stays correct, safe, and reliable, even when multiple users or 
systems interact with the database at the same time.

---

- 🏠 [Home](./../../README.md)
- 📚 [PostgreSql Tutorials](./../tutorials.md)
- 🧪 [ACID](./0_Understanding_ACID_in_PostgreSQL.md) 
- 🔢 [Transaction Identifiers](./2_transaction_identifiers.md)