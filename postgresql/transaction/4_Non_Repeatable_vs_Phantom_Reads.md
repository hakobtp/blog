# Non-Repeatable vs Phantom Reads: What's the Real Difference?

```info
Author      Ter-Petrosyan Hakob
```

---

In transactional database systems like **PostgreSQL**, **data consistency** and **concurrency** are essential. Two common problems that can happen when multiple transactions access the same data at the same time are **non-repeatable reads** and **phantom reads**.

These issues are directly related to how data is read and modified within a transaction and are influenced by the **isolation level** chosen.

In this post, we'll clearly explain the difference between these two anomalies using real PostgreSQL examples.

In this post, we'll break down the difference between these two read anomalies using simple language and practical PostgreSQL SQL examples.

## Non-Repeatable Read

A **non-repeatable** read happens when a transaction reads the **same row more than once**, but gets different results because another transaction updated or deleted that row in the meantime.

**Example scenario:**

Let's say we have the following table:

```sql
CREATE TABLE accounts (
    id SERIAL PRIMARY KEY,
    account_number VARCHAR(20) UNIQUE,
    balance INTEGER
);

INSERT INTO accounts (account_number, balance) VALUES ('1001', 500);

```

Now, imagine two concurrent transactions:

```sql
-- Transaction 1

BEGIN;

SELECT * FROM accounts WHERE account_number = '1001';

 id | account_number | balance 
----+----------------+---------
  1 | 1001           |     500
(1 row)

```

```sql

-- Transaction 2
BEGIN;

UPDATE accounts SET balance = balance + 100 WHERE account_number = '1001';

COMMIT;

```

```sql

-- Transaction 1

SELECT * FROM accounts WHERE account_number = '1001';

 id | account_number | balance 
----+----------------+---------
  1 | 1001           |     600
(1 row)



```

**What Happened?**
- **Transaction 1** starts and reads the balance of account '1001'. It sees 500.
- **Transaction 2** starts and updates the balance to 600, then commits.
- **Transaction 1** reads the same row again and sees 600, not the original 500.

This is a **non-repeatable read** — the same row returned **different values** within one transaction.

## Phantom Read

A **phantom read** occurs when a transaction **runs the same query twice**, and the **number of rows returned changes**, 
because another transaction inserted or deleted rows that match the query condition.

**Example scenario:**

We have a table like this:

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    order_number VARCHAR(20) UNIQUE,
    total_amount INTEGER
);

INSERT INTO orders (order_number, total_amount) VALUES ('1001', 100);
```

Then two transactions run:

```sql
-- Transaction 1
BEGIN;

SELECT * FROM orders WHERE total_amount > 90;

 id | order_number | total_amount 
----+--------------+--------------
  1 | 1001         |          100
(1 row)

```

```sql
-- Transaction 2
BEGIN;

INSERT INTO orders (order_number, total_amount) VALUES ('1002', 120);

COMMIT;
```

```sql
--- Transaction 1
SELECT * FROM orders WHERE total_amount > 90;

 id | order_number | total_amount 
----+--------------+--------------
  1 | 1001         |          100
  2 | 1002         |          120
(2 rows)

```

What Happened?
- **Transaction 1** runs a query to find all orders where total_amount > 90. It sees order '1001'.
- **Transaction 2** inserts a new order '1002' with a total_amount of 120, and commits.
- **Transaction 1** runs the same query again and now sees two rows: '1001' and '1002'.

This is a **phantom read** — the result set changed between two identical queries during the same transaction.

## Summary

|Feature                    |Non-Repeatable Read                                |Phantom Read                                               |
|:--------------------------|:--------------------------------------------------|:----------------------------------------------------------|
|Affects                    |Existing rows                                      |New rows (inserts or deletes)                              |
|When it happens            |Same row is read twice, returns different values   |Same query run twice returns a different number of rows    |
|Example change	            |`UPDATE` or `DELETE` of a row	                    |`INSERT` or `DELETE` of rows matching a condition          |
|Isolation level to avoid   |`REPEATABLE READ` or higher                        |`SERIALIZABLE`                                               |

---

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)
- [Multi-Version Concurrency Control](./3_Multi_Version_Concurrency_Control.md)
- [Savepoints](./5_Savepoints.md)


