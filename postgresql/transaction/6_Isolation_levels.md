# 🛡️ Isolation levels

```info
Author      Ter-Petrosyan Hakob
```
---

In PostgreSQL—as in many relational database management systems (RDBMS)—isolation levels control how transactions interact with each other and with the data they access. Different isolation levels offer trade-offs between consistency, concurrency, and performance by determining which changes become visible to concurrent transactions.

In this post, we explain each isolation level, discuss common concurrency anomalies, and show practical examples of setting isolation levels in PostgreSQL.

## Concurrency Anomalies in Transactions

Understanding the anomalies that isolation levels aim to address can help in choosing the right one for your application:

- **Dirty Reads:** Occur when a transaction reads data that another transaction has written but not yet committed. No production database allows dirty reads.

**Non-Repeatable Reads:** Happen when a transaction reads the same row twice and sees different data because another committed transaction updated that row between the two reads.

**Phantom Reads:** Occur when a transaction re-executes a query returning a set of rows and finds additional rows (or missing rows) that were inserted or deleted by another committed transaction.

Higher isolation levels (i.e., **REPEATABLE READ** and **SERIALIZABLE**) are designed to eliminate these anomalies—but with the trade-off of potentially reduced concurrency and performance.

## Overview of Isolation Levels

The SQL standard defines four isolation levels, each increasing the level of protection against anomalies:

- **Read Uncommitted** 
    - Lowest level. In theory, it allows transactions to see uncommitted changes (dirty reads). However, PostgreSQL does not 
        actually implement this level—requests for **READ UNCOMMITTED** are silently mapped to **READ COMMITTED**.

- **Read Committed**
    - Default level. A transaction sees only data committed before each individual statement begins. This prevents 
        dirty reads but does not guarantee consistent results within a multi-statement transaction (non-repeatable or phantom reads may occur).

- **Repeatable Read**
    - This level guarantees that all statements in a transaction see a consistent snapshot taken at the transaction’s start. 
        It prevents non-repeatable reads but can still allow phantom reads in some systems. (In PostgreSQL, the snapshot `“freezes”` the data seen by the transaction as of its first query.)

- **Serializable**
    - The highest isolation level forces transactions to execute as if they were run one after the other. It prevents dirty reads, non-repeatable reads, and phantom reads. While ensuring the strictest consistency, **SERIALIZABLE** transactions may be aborted if concurrent transactions conflict, requiring a retry.

## Configuring Transaction Isolation in PostgreSQL

You can set the isolation level for a transaction at the beginning of the transaction block. For example:

```sql
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;

```

Or, omitting the optional keyword for brevity:

```sql
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

```

❗ **Important:** The **SET TRANSACTION** statement must be the first command in the transaction block. Any statement executed before setting the isolation level causes subsequent attempts to change it to fail:

---

## 📌 Explore More

- 🏠 [Home](./../../README.md)
- 📚 [PostgreSql Tutorials](./../tutorials.md)
- 💾 [Savepoints](./5_Savepoints.md)