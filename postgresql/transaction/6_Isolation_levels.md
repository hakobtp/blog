# Isolation levels

```info
Author      Ter-Petrosyan Hakob
```
---

In PostgreSQL—as in many relational database management systems (RDBMS)—isolation levels control how transactions interact with each other and with the data they access. Different isolation levels offer trade-offs between consistency, concurrency, and performance by determining which changes become visible to concurrent transactions.

In this post, we explain each isolation level, discuss common concurrency anomalies, and show practical examples of setting isolation levels in PostgreSQL.

## Concurrency Anomalies in Transactions

Understanding the anomalies that isolation levels aim to address can help in choosing the right one for your application:

- **Dirty Reads:** Occur when a transaction reads data that another transaction has written but not yet committed. No production database allows dirty reads.

- **Non-Repeatable Reads:** Happen when a transaction reads the same row twice and sees different data because another committed transaction updated that row between the two reads.

- **Phantom Reads:** Occur when a transaction re-executes a query returning a set of rows and finds additional rows (or missing rows) that were inserted or deleted by another committed transaction.

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

> **IMPORTANT:** 
>
>The **SET TRANSACTION** statement must be the first command in the transaction block. 
> Any statement executed before setting the isolation level causes subsequent attempts to change it to fail:

```sql
BEGIN;
SELECT count(*) FROM categories;

-- a query has been executed, the SET TRANSACTION
-- is not anymore the very first command

SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
ERROR:  SET TRANSACTION ISOLATION LEVEL must be called before any query
```

Once the transaction has started, the isolation level cannot be changed.

## In-Depth: Isolation Levels in PostgreSQL

**READ UNCOMMITTED**
- **Behavior:** Although defined by the SQL standard, PostgreSQL does not truly implement **READ UNCOMMITTED**. 
    Any attempt to set this level is silently upgraded to **READ COMMITTED**.
- **Implication:** There is no risk of dirty reads because PostgreSQL always enforces at least **READ COMMITTED**.

**READ COMMITTED**
- **Behavior:** This is the default isolation level. Each statement in a transaction sees only data that was committed before the statement began. That means each query may see a more recent state of the database if other transactions commit changes in the meantime.

- **Benefits/Limitations:**
    - **Prevents:** Dirty reads.
    - **May Allow:** Non-repeatable and phantom reads.

**REPEATABLE READ**
- **Behavior:** All statements in a transaction see a consistent snapshot taken when the transaction started (or with the first query). This guarantees that repeated reads produce the same result—provided no explicit writes are performed within the transaction.
- **Benefits/Limitations:**
    - **Prevents:** Non-repeatable reads.
    - **May Allow:** Phantom reads in some systems. In PostgreSQL, the snapshot ensures consistency during the transaction but note that not every case of phantom behavior is completely eliminated.


**SERIALIZABLE**
- **Behavior:** Transactions execute as if they were serialized (run one after another). Even if they execute concurrently, PostgreSQL checks for conflicts and may abort one transaction if the concurrent executions would have produced different results than some serial order.

    **Example Scenario:**
    ```sql
    -- Session 1:
    BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
    UPDATE categories SET name = lower(name);
    ```

    ```sql
    -- Session 2:

    BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
    UPDATE categories SET name = '[' || name || ']';
    -- This session gets blocked waiting for Session 1 to finish.
    ```

    If Session 1 commits:
    
    ```sql
    -- Session 1:
    COMMIT;

    ```
    Session 2 later attempting to continue:

    ```sql
    UPDATE categories SET name = '[' || name || ']';
    -- ERROR: could not serialize access due to concurrent update
    ```

    Even when transactions modify different tuples, a conflict might be detected at commit time:
    ```sql
    -- Session 1:
    BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
    UPDATE categories SET name = '{' || name || '}' WHERE name = 'java';
    ```

    ```sql
    -- Session 2:
    BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
    UPDATE categories SET name = '[' || name || ']' WHERE name = 'perl';
    ```

   This time, there is no locking of the second transaction because the touched tuples are completely different. However, as soon as the first transaction executes a **COMMIT** , the second transaction is no longer able to **COMMIT** by itself:
    
    ```sql
    -- Session1 
    COMMIT;

    -- Session 2:
    COMMIT;

    ERROR:  could not serialize access due to read/write dependencies among transactions
    DETAIL:  Reason code: Canceled on identification as a pivot, during commit attempt.
    HINT:  The transaction might succeed if retried
    ```
- **Benefits/Limitations:**
    - **Prevents:** Dirty reads, non-repeatable reads, and phantom reads.
    - **Implication:** Transactions may need to be retried in the case of serialization failures.    

## Phantom Reads: A Closer Look

Although discussed above, phantom reads deserve special attention:

- **Definition:** A phantom read occurs when a transaction executes a query twice and, on the second execution, finds additional rows (or missing rows) because another transaction inserted or deleted rows that meet the query’s criteria.

- **Example Scenario:**
    1. `Transaction A` queries the Employees table for records with `Gender = 'Male'` and retrieves 2 rows.<br>
    2. Concurrently, `Transaction B` inserts a new row for a `male` employee.<br>
    3. When `Transaction A` re-executes the query, it now returns 3 rows, showing the `“phantom”` row.<br>
- **Mitigation:** Using **SERIALIZABLE** (or, in some systems, snapshot isolation) prevents phantom reads by ensuring the transaction works from a consistent snapshot of the data.

## Unrepeatable Reads vs. Phantom Reads

Both anomalies relate to changes observed during a transaction:

- **Unrepeatable Reads:** Occur when the same row is read twice within a transaction and the data has changed due to another committed transaction.
- **Phantom Reads:** Involve a change in the overall result set—the same query executed twice returns a different number of rows because of concurrent insertions or deletions.

By choosing the appropriate isolation level, you can balance the need for consistency with performance:

- **READ COMMITTED** is ideal for high concurrency when non-repeatable or phantom reads are acceptable.
- **REPEATABLE READ** and **SERIALIZABLE** offer stronger guarantees for applications that require strict data consistency.

---

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)
- [Savepoints](./5_Savepoints.md)
- [Deadlocks](./7_Deadlocks.md)