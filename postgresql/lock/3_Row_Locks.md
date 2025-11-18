# Row Locks

Table locks control access to entire tables, but PostgreSQL also uses row locks to protect individual rows during updates and deletes.

These locks ensure that two transactions don’t modify the same row simultaneously — preventing lost updates and maintaining data consistency.

**Important:**

- Newly inserted rows don’t need explicit locks — they belong exclusively to the inserting transaction until it commits.
- Row locks are stored inside each row’s metadata (the **xmax** field), not just in memory.

There are four main types of row-level locks:

## FOR UPDATE

`FOR UPDATE` is the strongest type of row-level lock. When a row is locked with `FOR UPDATE`, no other transaction can 
modify it or acquire another `FOR UPDATE` lock on the same row until the current transaction finishes.

It’s automatically applied by the following commands:

```sql
UPDATE
DELETE
SELECT ... FOR UPDATE
```

Let’s see how it works in practice:

```sql
-- Transaction 1
BEGIN;
SELECT * FROM products WHERE id = 5 FOR UPDATE;

-- This row is now locked.
-- No other transaction can update or delete it until Transaction 1 finishes.
```

Meanwhile, if another transaction tries this:

```sql
-- Transaction 2
BEGIN;
UPDATE products SET price = price + 10 WHERE id = 5;
```

It will wait until `Transaction 1` either commits or rolls back.

Then, when `Transaction 1` finishes:

```sql
COMMIT;  -- or ROLLBACK;
```

`Transaction 2` will automatically continue and acquire the lock.

If you want the second transaction not to wait but instead skip locked rows, you can use:

### FOR UPDATE with SKIP LOCKED

The `SKIP LOCKED` option changes this behavior. Instead of waiting for locked rows, PostgreSQL skips them and returns only those currently available for locking.

This is particularly useful in job queues, worker pools, or parallel processing systems where multiple workers process tasks concurrently without blocking each other.

```sql
BEGIN;

-- Select and lock the first available pending order
SELECT * 
FROM orders
WHERE status = 'pending'
FOR UPDATE SKIP LOCKED
LIMIT 1;
```

If several workers run this query at the same time:

- Each worker locks a different row.
- Locked rows are skipped, so workers don’t block each other.
- Once a worker finishes processing a task, it can update the row, commit, and release the lock.

| Feature                  | Behavior                                                                       |
| ------------------------ | ------------------------------------------------------------------------------ |
| `FOR UPDATE`             | Locks the selected rows; other transactions **wait** if they try to lock them. |
| `FOR UPDATE SKIP LOCKED` | Locks only available rows; **skips** already locked ones — no waiting.         |
| **Use case**             | Job queues, batch processing, and avoiding deadlocks in concurrent workers.    |

**Example in a Job Queue:**

```sql
-- Worker 1
BEGIN;
SELECT * FROM tasks
WHERE status = 'pending'
FOR UPDATE SKIP LOCKED
LIMIT 1;

-- Worker 2 runs at the same time
-- It skips the row locked by Worker 1
```

Both workers safely obtain different rows without waiting or blocking each other.
This allows your system to process multiple jobs efficiently in parallel.

## FOR NO KEY UPDATE

`FOR NO KEY UPDATE` is used when you need to update a row but not any of its key columns (columns that are part of a primary key or unique constraint).
It’s weaker than `FOR UPDATE`, meaning it places a less restrictive lock on the row. Other transactions can still acquire a `FOR KEY SHARE` lock on the same row, but not stronger ones like` FOR UPDATE`.

Use `FOR NO KEY UPDATE` when:
- You are updating non-key columns (for example, status, description, or price),
- And you want to avoid unnecessarily blocking other transactions that only need to read key columns.

Consider the following table:

```sql
CREATE TABLE accounts (
    id SERIAL PRIMARY KEY,
    balance NUMERIC,
    last_accessed TIMESTAMP
);
```

Now you want to update only the `last_accessed` time, not the primary key (`id`):

```sql
BEGIN;

-- Lock the row but allow other sessions to read key columns
SELECT * 
FROM accounts
WHERE id = 10
FOR NO KEY UPDATE;

-- Then update a non-key column
UPDATE accounts
SET last_accessed = NOW()
WHERE id = 10;

COMMIT;
```

Here’s what happens:

- The row with `id = 10` is locked for non-key updates.
- Other transactions can still read the row using `FOR KEY SHARE`.
- But no other transaction can update this row until your transaction finishes.

**Real-World Example: Logging Last Access Time**

Imagine multiple users checking their account balances simultaneously.
Each query wants to update only the `last_accessed` column.

Using `FOR UPDATE` would be too strict — it would block all other transactions.
But `FOR NO KEY UPDATE` lets each transaction safely update timestamps without interfering with others reading account data.

| Feature                  | Behavior                                                                            |
| ------------------------ | ----------------------------------------------------------------------------------- |
| `FOR NO KEY UPDATE`      | Locks selected rows but allows `FOR KEY SHARE`; used when updating non-key columns. |
| **Use case**             | Updating metadata, timestamps, or counters without touching key columns.            |
| **Command that uses it** | `UPDATE` (implicitly applies `FOR NO KEY UPDATE`).                                  |

## FOR SHARE

`FOR SHARE` places a shared lock on the selected rows.
This means multiple transactions can hold this lock on the same row at the same time, but none of them can modify the locked row until all shared locks are released.

It’s commonly used when you need to read data that might soon be updated, while preventing other transactions from changing it during your read.

**Behavior:**
- Several transactions can safely read the same rows using `FOR SHARE`.
- No transaction can perform an `UPDATE` or `DELETE` on those rows until all shared locks are released.
- However, other transactions can still take another `FOR SHARE` or `FOR KEY SHARE` lock on the same rows.

```sql
BEGIN;

-- Lock the product row in shared mode
SELECT * 
FROM products
WHERE id = 5
FOR SHARE;

-- Do some checks or calculations here
-- ...

COMMIT;
```

While this transaction runs:

- Other transactions can also execute `SELECT ... FOR SHARE` on the same row.
- But if another transaction tries to run:

```sql
UPDATE products SET price = price + 10 WHERE id = 5;
```

It will wait until the first transaction finishes.

**Real-World Example: Reading Data Before Update**

Imagine a system where analysts need to read product details before approving price changes.
Each analyst can use `FOR SHARE` to ensure that no one updates the row while it’s being read.
Once they finish their analysis, they release the lock, allowing updates to continue.

| Feature                  | Behavior                                                    |
| ------------------------ | ----------------------------------------------------------- |
| `FOR SHARE`              | Allows multiple readers; prevents any updates or deletes.   |
| **Use case**             | Safe concurrent reading before making updates or decisions. |
| **Command that uses it** | `SELECT ... FOR SHARE`                                      |

## FOR KEY SHARE

`FOR KEY SHARE` is the weakest type of row-level lock in PostgreSQL.
It allows transactions to read rows safely while still permitting non-key updates by other transactions.
However, it prevents changes to key columns (such as primary or unique keys) and blocks deletions of the locked rows.

This lock is often used when you want to ensure that a referenced key value remains stable, especially when working with foreign key relationships.

**Behavior:**

- Multiple transactions can hold a `FOR KEY SHARE` lock on the same rows simultaneously.
- Other transactions can still update non-key columns of those rows.
- But no transaction can delete the row or modify its key columns (like a primary key).
- It’s the weakest and most permissive of all `SELECT ... FOR` locks.

Consider the following tables:

```sql
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name TEXT,
    loyalty_points INT
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(id),
    total NUMERIC
);

```

Now suppose you want to read customer data before creating a new order and ensure the customer record can’t be deleted during that process:

```sql
BEGIN;

-- Lock the customer row so that it can't be deleted,
-- but allow non-key updates like changing loyalty_points
SELECT * 
FROM customers
WHERE id = 10
FOR KEY SHARE;

-- Safely insert a new order for that customer
INSERT INTO orders (customer_id, total) VALUES (10, 250);

COMMIT;
```

Here’s what happens:

- The row in customers with `id = 10` is locked with `FOR KEY SHARE`.
- Another transaction can still run:

```sql
UPDATE customers SET loyalty_points = loyalty_points + 1 WHERE id = 10;
```

because this update doesn’t change a key column.

But it cannot execute:

```sql
DELETE FROM customers WHERE id = 10;
```

because `FOR KEY SHARE` blocks deletes and key changes.

| Feature                  | Behavior                                                                      |
| ------------------------ | ----------------------------------------------------------------------------- |
| `FOR KEY SHARE`          | Weakest row lock. Allows non-key updates but blocks key changes and deletes.  |
| **Use case**             | Reading rows referenced by foreign keys; ensuring key stability during reads. |
| **Command that uses it** | `SELECT ... FOR KEY SHARE`                                                    |

## Summary Table

| Lock Type         | Allows SELECT | Allows UPDATE    | Allows DELETE |
| ----------------- | ------------- | ---------------- | ------------- |
| FOR KEY SHARE     | ✅             | ✅ (non-key only) | ❌             |
| FOR SHARE         | ✅             | ❌                | ❌             |
| FOR NO KEY UPDATE | ✅             | ✅                | ❌             |
| FOR UPDATE        | ✅             | ❌ (by others)    | ❌             |


---

| Lock Type           | Strength     | Description                                                                      |
| ------------------- | ------------ | -------------------------------------------------------------------------------- |
| `FOR UPDATE`        | Strongest    | Locks the entire row — no other lock can be taken.                               |
| `FOR NO KEY UPDATE` | Weaker       | Locks the row for updates that don’t change key columns; allows `FOR KEY SHARE`. |
| `FOR SHARE`         | Weaker still | Used for read-only operations; prevents updates but allows other readers.        |
| `FOR KEY SHARE`     | Weakest      | Allows reading key values even when `FOR NO KEY UPDATE` is held.                 |


## Page Locks — Protecting Memory Pages

A PostgreSQL page is an 8 KB block of memory that holds tuples (rows).
Because PostgreSQL uses a process-based architecture (each connection gets its own backend process), it needs internal locks to control access to these pages.

- Multiple processes can read the same page simultaneously.
- But only one process can write to it at a time.

This mechanism works like mutexes or semaphores in operating systems — it ensures data integrity when many processes attempt to access the same memory area.

## Deadlocks — When Locks Block Each Other

A deadlock occurs when two transactions each hold locks that the other needs.
Neither can continue, so PostgreSQL detects the situation and cancels one of them.

```sql
-- Transaction 1
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- Transaction 2
BEGIN;
SELECT * FROM accounts WHERE id = 2 FOR UPDATE;

-- Tx1 now tries to update row 2
UPDATE accounts SET balance = balance - 100 WHERE id = 2;

-- Tx2 tries to update row 1
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
```

Both transactions wait for each other indefinitely — PostgreSQL detects the deadlock and aborts one of them

**Tip:** Keep transactions short, and always acquire locks in a consistent order to avoid deadlocks.

---

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)
- [Table Locks](./2_Table_Locks.md)