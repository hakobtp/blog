# Table Locks

When I first started learning about databases, I believed there were only two kinds of locks: **shared** and **exclusive**.

- A **shared lock** (or read lock) means that many people can read the same data at the same time.
- An **exclusive lock** (or write lock) means that only one process can make changes — everyone else must wait.

It sounded simple. But when I started exploring PostgreSQL, I discovered that the real picture is far more detailed and powerful. 
PostgreSQL has five lock categories and over a dozen individual lock types, each with its own purpose and behavior.

In this post, we’ll explore:

- How PostgreSQL manages table and row locks.
- What commands take which locks.
- Which locks block others.
- And finally, why understanding this helps you write better, faster, and safer SQL.

Let’s begin our journey into PostgreSQL’s world of locks.

## Table Locks — Controlling Access to Tables

If you asked me a few years ago what a table lock was, I might have said: “It’s a lock on a table that stops anyone else from using it.”

That sounds logical but isn’t true in PostgreSQL. In reality, PostgreSQL has eight table lock types, and a transaction can hold more than one lock on the same table. Some locks allow other operations to continue; others block everything.

When working with PostgreSQL, you might occasionally check the locks held on your database tables using the `pg_locks` system view. For example, consider the following query:

```sql

SELECT pid, relation::regclass, mode, granted
FROM pg_locks
WHERE relation IS NOT NULL;

```

Running this query could return a result like this:

```bash
24088 | pg_locks | AccessShareLock | true
```
At first, this can look confusing. Let’s explain each part:

- **pid (24088):** This is the process ID of the session holding the lock. Here, backend `24088` has a lock on the `pg_locks` table.
- **relation::regclass (pg_locks):** This shows which table is locked. In this case, it is the `pg_locks` table itself.
- **mode (AccessShareLock):** PostgreSQL has many types of locks. `AccessShareLock` is the least restrictive. It happens automatically 
    when you read a table with `SELECT`. It usually does not block other queries, except very exclusive operations like `DROP TABLE` or `ALTER TABLE`.
- **granted (true):** This means the lock is currently held, not just requested.

This output is completely normal. When you run a `SELECT` on `pg_locks`, PostgreSQL automatically acquires an `AccessShareLock` on the table.
- This type of lock is harmless.
- It does not block other queries, except those requiring exclusive access, such as `DROP TABLE` or `ALTER TABLE`.
- Essentially, `pg_locks` is showing the lock that your own query implicitly holds on the system catalog.

Sometimes, you might encounter queries that are genuinely blocked by other sessions. PostgreSQL provides a way to find these cases. 
Here’s a query that helps identify blocked and blocking sessions:

```sql
SELECT blocked_locks.pid         AS blocked_pid,
       blocked_activity.usename  AS blocked_user,
       blocked_activity.query    AS blocked_query,
       blocking_locks.pid        AS blocking_pid,
       blocking_activity.usename AS blocking_user,
       blocking_activity.query   AS blocking_query,
       blocked_locks.mode        AS blocked_mode,
       blocking_locks.mode       AS blocking_mode
FROM pg_locks blocked_locks
         JOIN pg_stat_activity blocked_activity
              ON blocked_activity.pid = blocked_locks.pid
         JOIN pg_locks blocking_locks
              ON blocking_locks.locktype = blocked_locks.locktype
                  AND blocking_locks.database IS NOT DISTINCT FROM blocked_locks.database
                  AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
                  AND blocking_locks.page IS NOT DISTINCT FROM blocked_locks.page
                  AND blocking_locks.tuple IS NOT DISTINCT FROM blocked_locks.tuple
                  AND blocking_locks.pid != blocked_locks.pid
         JOIN pg_stat_activity blocking_activity
              ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.granted;
```

- **blocked_pid** session waiting for a lock
- **blocking_pid** session currently holding the lock
- **blocked_query / blocking_query** shows which queries are involved
- **blocked_mode / blocking_mode** type of locks involved

Using this, you can quickly identify the queries causing blocks and take action, such as terminating a long-running session or optimizing a transaction to avoid conflicts.


## ACCESS EXCLUSIVE: The Strongest Lock

This is the most restrictive lock in PostgreSQL. When a transaction holds an `ACCESS EXCLUSIVE` lock, nobody else can even read from the table. 
It blocks all other table lock types.

When this lock is active:

- You cannot `SELECT`, `INSERT`, `UPDATE`, or `DELETE`.
- You cannot change the table structure.
- Even maintenance commands like `VACUUM` or `ANALYZE` must wait.

Typical commands that acquire this lock include:

```sql
DROP TABLE
TRUNCATE
VACUUM FULL
REINDEX
CLUSTER
ALTER TABLE ADD COLUMN
ALTER TABLE DROP COLUMN
ALTER TABLE SET DATA TYPE
ALTER TABLE RENAME
REFRESH MATERIALIZED VIEW
```

**Example: ACCESS EXCLUSIVE Lock in Action**

Suppose we have a table called `users`. Now, let’s see what happens when we acquire an `ACCESS EXCLUSIVE` lock.

This lock is often used when PostgreSQL needs to rebuild or reorganize the physical data on disk, which must be done in isolation.

Start a session and run `VACUUM FULL`
```sql
-- Session 1:
-- This will acquire an ACCESS EXCLUSIVE lock on the users table
VACUUM FULL users;
```

At this moment:

- PostgreSQL locks the users table in `ACCESS EXCLUSIVE` mode.
- No other session can read or write to users until `VACUUM FULL` finishes.

```sql
-- Session 2:
SELECT * FROM users;
```

What happens:
- This query blocks (waits) because the table is locked with `ACCESS EXCLUSIVE`.
- PostgreSQL won’t let you read until the lock is released.

Try to insert into the table in another session

```sql
-- Session 2:
INSERT INTO users(name, email) VALUES ('Alice', 'alice@example.com');
```

What happens:
- This query also blocks.
- No writes, no structure changes, no `SELECT`s are allowed until `Session 1` finishes `VACUUM FULL`.

Once the `VACUUM FULL` in `Session 1` completes:

- The lock is released.
- All blocked queries in `Session 2` will continue and execute normally.


**Key takeaway:** 
- `ACCESS EXCLUSIVE` locks are extremely strong. They are needed for 
    operations that rebuild or change the table’s structure,  like `VACUUM FULL`, `DROP TABLE`, `ALTER TABLE`, or `CLUSTER`.


## ACCESS SHARE: The Lightest Lock

The `ACCESS SHARE` lock is the lightest and safest lock in PostgreSQL. It is acquired by commands that **only read data** and **do not modify the table**.

Commands that take this lock

```sql
SELECT * FROM users;
COPY users TO '/path/to/file.csv' WITH (FORMAT csv, HEADER);
```

> **Note:** `COPY TO` is a standalone command, not part of `SELECT`. It exports table data to a file and acquires an `ACCESS SHARE` lock.

How it behaves

- **ACCESS SHARE** only conflicts with **ACCESS EXCLUSIVE**.
- If a `VACUUM FULL`, `DROP TABLE`, or `ALTER TABLE` is running, your `SELECT` or `COPY TO` will wait.
- Otherwise, normal reads and writes can run simultaneously without blocking.

> **Example:** You can run hundreds of `SELECT` queries on a table at the same time — they all share the table peacefully.

**Example: ACCESS SHARE Lock in Action**

```sql
-- Session 1
SELECT * FROM users;
```

- Table is locked in `ACCESS SHARE` mode.
- This does not block other readers or normal writes.

```sql
-- Session 2
SELECT * FROM users;
```

Result:
- Query executes immediately.
- Multiple reads happen concurrently.
- This demonstrates why `ACCESS SHARE` is called the lightest lock.

```sql
-- Session 3
INSERT INTO users(name, email) VALUES ('Charlie', 'charlie@example.com');
```

Result:
- `INSERT` executes immediately.
- `ACCESS SHARE` does not block writes. Normal DML operations (`INSERT`/`UPDATE`/`DELETE`) are allowed alongside readers.

```sql
-- Session 4
VACUUM FULL users;
```

Result:
- `VACUUM FULL` waits until all `ACCESS SHARE` locks are released.
- `SELECT` queries in Sessions 1 and 2 block the `ACCESS EXCLUSIVE` operation until they finish.

- **Key Takeaways:**
- `ACCESS SHARE` is used for reading only (`SELECT`, `COPY TO`).
- Multiple readers can coexist without blocking each other.
- Conflicts occur only with `ACCESS EXCLUSIVE` operations like `VACUUM FULL`, `DROP TABLE`, or `ALTER TABLE`.
- Ideal for reporting, analytics, and dashboards that do not modify data.


## EXCLUSIVE : Reading Allowed, Writing Blocked

The EXCLUSIVE lock is similar to ACCESS EXCLUSIVE but more relaxed. It allows other sessions to read the table but blocks modifications.

Only one major command uses this lock:

REFRESH MATERIALIZED VIEW CONCURRENTLY

This design lets PostgreSQL update a materialized view while other users can still query it — a balance between consistency and concurrency.


The `EXCLUSIVE` lock is a little weaker than `ACCESS EXCLUSIVE`. It stops changes to the table but other sessions can still read the data. 
This lock is useful when you want to update a table safely without stopping users from reading it.

Only a few commands use this lock. The most common one is:

```sql
REFRESH MATERIALIZED VIEW CONCURRENTLY;
```

This command updates a materialized view while letting other sessions read it at the same time.

How it works

- Other sessions can read the table (`SELECT` works).
- Other sessions cannot write to the table (`INSERT`, `UPDATE`, `DELETE` are blocked).
- Only one session can hold an `EXCLUSIVE` lock at a time.

> **Why it exists:** This lock allows updates while letting users read data. It keeps a good balance between consistency and concurrency.

**Example: EXCLUSIVE Lock in Action**

Suppose we have a materialized view called `user_stats`.

```sql
-- Session 1
REFRESH MATERIALIZED VIEW CONCURRENTLY user_stats;
```

At this moment:

PostgreSQL locks the view in `EXCLUSIVE` mode.
Other users can still read from the view without waiting.

```sql
-- Session 2
SELECT * FROM user_stats;
```

Result:
- The query runs immediately.
- Reading is not blocked.

```sql
-- Session 3
INSERT INTO user_stats(user_id, login_count) VALUES (1, 10);
```

Result:

- This query waits until Session 1 finishes.
- Writing is blocked while the lock is active.

Once `REFRESH MATERIALIZED VIEW CONCURRENTLY` finishes:
- The `EXCLUSIVE` lock is released.
- All blocked writes can now continue.

**Key Takeaways:**
- `EXCLUSIVE` locks allow reading but block writing.
- Useful for refreshing materialized views without stopping readers.
- Balances data consistency with concurrent access.

## ROW SHARE: For SELECT ... FOR Commands

`ROW SHARE` locks are used when a query reads rows but may change them later.

Commands that take this lock

```sql
SELECT ... FOR UPDATE
SELECT ... FOR SHARE
SELECT ... FOR KEY SHARE
SELECT ... FOR NO KEY UPDATE
```

How it works

- This lock conflicts only with `ACCESS EXCLUSIVE` and `EXCLUSIVE` locks.
- You can still read the table normally.
- You cannot make changes to the table structure (like `ALTER TABLE` or `DROP TABLE`) while the lock is active.

Suppose your app runs:

```sql
SELECT * FROM orders WHERE id = 10 FOR UPDATE;
```

- PostgreSQL locks that specific row.
- It also takes a table-level `ROW SHARE` lock.
- Other users cannot change that row until your transaction finishes.

> This is useful when you want to read a row and then safely update it, without blocking other readers.


## ROW EXCLUSIVE: For DML Operations

`ROW EXCLUSIVE` locks are used for **data-changing commands** — the ones that **modify rows**.

Commands that take this lock
```sql
INSERT
UPDATE
DELETE
MERGE
COPY FROM
```

> Even if your `UPDATE` changes zero rows, PostgreSQL still takes the lock.

How it works
- Multiple transactions can hold this lock at the same time.
- Stronger locks, like `ACCESS EXCLUSIVE` (used by `ALTER TABLE` or `VACUUM FULL`), must wait until all `ROW EXCLUSIVE` locks are finished.

Suppose a long-running transaction is inserting data:

```sql
-- Session 1
INSERT INTO orders(user_id, total) VALUES (1, 100);
```

If another session tries to run:

```sql
ALTER TABLE orders ADD COLUMN status TEXT;
```

- This query waits until the `INSERT` finishes.
- That’s because `ALTER TABLE` requires a stronger lock than `ROW EXCLUSIVE`.

> `ROW EXCLUSIVE` is for normal write operations. It lets multiple writers work at the same time, but big table changes must wait.

## SHARE ROW EXCLUSIVE: Block Writers, Allow Readers

`SHARE ROW EXCLUSIVE` locks stop data changes but still allow reading, including `SELECT FOR` queries.

Commands that take this lock

```sql
CREATE TRIGGER
ALTER TABLE ADD FOREIGN KEY
ALTER TABLE ENABLE TRIGGER
ALTER TABLE DISABLE TRIGGER
```

How it works
- Other sessions can read the table while this lock is active.
- Data changes (`INSERT`, `UPDATE`, `DELETE`) are blocked.
- Only one session can hold this lock at a time — if another session tries the same operation, it must wait.

This lock is useful when you want to make structural changes that don’t stop users from reading, but you cannot allow other writers at the same time.

## SHARE: For Creating Indexes

`SHARE` locks stop data changes but allow multiple sessions to hold the same lock at the same time.

Commands that take this lock
```sql
CREATE INDEX
```

How it works
- While building an index, PostgreSQL reads all table rows, so data changes (`INSERT`, `UPDATE`, `DELETE`) are blocked.
- Several `CREATE INDEX` commands can run at the same time on the same table, as long as they don’t try to change data.

> If you want to build an index without blocking writes, use:

```sql
CREATE INDEX CONCURRENTLY;
```

This lets users `read` and `write` while the index is being created.

## SHARE UPDATE EXCLUSIVE: For Maintenance Tasks

`SHARE UPDATE EXCLUSIVE` locks allow normal reading and writing but stop structural changes and some maintenance operations.

Commands that take this lock

```sql
VACUUM
CREATE INDEX CONCURRENTLY
ANALYZE
COMMENT ON
REINDEX CONCURRENTLY
ALTER TABLE VALIDATE CONSTRAINT
ALTER TABLE DETACH PARTITION
```

How it works

- Other sessions can **read** and **write** data normally.
- Structural changes (like `ALTER TABLE`) or some maintenance operations are blocked.
- This lock is often used by background tasks, like `VACUUM`, to clean up old rows without stopping normal activity.

 It is useful for maintenance operations that must run safely while users continue to read and write data.
 
## Table Lock Matrix (Simplified)

Here’s an easier way to think about lock conflicts:

| Lock Type              | Blocks Reads | Blocks Writes | Blocks Schema Changes |
| ---------------------- | ------------ | ------------- | --------------------- |
| ACCESS SHARE           | ❌            | ❌             | ❌                     |
| ROW SHARE              | ❌            | ❌             | ✅                     |
| ROW EXCLUSIVE          | ❌            | ✅             | ✅                     |
| SHARE                  | ❌            | ✅             | ✅                     |
| SHARE ROW EXCLUSIVE    | ❌            | ✅             | ✅                     |
| SHARE UPDATE EXCLUSIVE | ❌            | ✅             | ✅                     |
| EXCLUSIVE              | ❌            | ✅             | ✅                     |
| ACCESS EXCLUSIVE       | ✅            | ✅             | ✅                     |


In short:
- The further down the list, the stronger the lock.
- Stronger locks block more operations.

---

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)