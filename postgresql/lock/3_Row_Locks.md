# Row Locks

Table locks control access to tables, but PostgreSQL also uses row locks to protect individual rows during 
updates and deletes. These locks ensure that two transactions don’t modify the same row at the same 
time — preventing lost updates and keeping your data consistent.

**Important:**
- Inserted rows don’t need locks — they belong to the inserting transaction until committed.
- Row locks are stored inside the row’s metadata (the **xmax** field), not just in memory.

There are four main row lock types:

## FOR UPDATE

`FOR UPDATE` is the strongest type of row-level lock. When a row is locked with `FOR UPDATE`, no other transaction can modify 
it or acquire another `FOR UPDATE` lock on the same row until the current transaction ends.

It is automatically used by the following commands

```sql
UPDATE
DELETE
SELECT ... FOR UPDATE
```

**Example:**

Let’s see how it works in practice:

```sql
-- Transaction 1
BEGIN;
SELECT * FROM products WHERE id = 5 FOR UPDATE;

-- Now this row is locked.
-- No other transaction can update or delete it until Transaction 1 ends.
```

Meanwhile, if another transaction tries this:

```sql
-- Transaction 2
BEGIN;
UPDATE products SET price = price + 10 WHERE id = 5;
```

It will wait until `Transaction 1` either commits or rolls back.

Then, when `Transaction 1` ends:
```sql
COMMIT;  -- or ROLLBACK;
```

`Transaction 2` will automatically continue and acquire the lock.

If you want the second transaction not to wait but instead skip locked rows, you can use:

```sql
SELECT * FROM products WHERE id = 5 FOR UPDATE SKIP LOCKED;
```

This is useful for job queues or task processing systems.

### FOR UPDATE with SKIP LOCKED

The `SKIP LOCKED` option changes this behavior.
Instead of waiting for locked rows, PostgreSQL skips them and returns only the rows that are currently free to lock. 
This is especially useful in job queues, worker pools, or parallel processing systems, where multiple workers process tasks concurrently without blocking each other.

**Example:**
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

| Feature                  | Behavior                                                                    |
| ------------------------ | --------------------------------------------------------------------------- |
| `FOR UPDATE`             | Locks the selected rows; other sessions **wait** if they try to lock them.  |
| `FOR UPDATE SKIP LOCKED` | Locks only available rows; **skips** already locked ones — no waiting.      |
| **Use case**             | Job queues, batch processing, and avoiding deadlocks in concurrent workers. |

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

Both workers safely get different rows without waiting or blocking each other.
This allows your system to process multiple jobs in parallel efficiently.


---

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)
- [Table Locks](./2_Table_Locks.md)