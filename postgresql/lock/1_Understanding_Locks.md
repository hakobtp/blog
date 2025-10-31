# Understanding Locks

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


## 1. ACCESS EXCLUSIVE — The Strongest Lock

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


**Key takeaway:** `ACCESS EXCLUSIVE` locks are extremely strong. They are needed for operations that rebuild or change the table’s structure, 
like `VACUUM FULL`, `DROP TABLE`, `ALTER TABLE`, or `CLUSTER`.


## 2. ACCESS SHARE — The Lightest Lock

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

---

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)