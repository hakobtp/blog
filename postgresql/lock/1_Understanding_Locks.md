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

**pid (24088):** This is the process ID of the session holding the lock. Here, backend `24088` has a lock on the `pg_locks` table.
- **relation::regclass (pg_locks):** This shows which table is locked. In this case, it is the `pg_locks` table itself.
- **mode (AccessShareLock):** PostgreSQL has many types of locks. `AccessShareLock` is the least restrictive. It happens automatically 
    when you read a table with `SELECT`. It usually does not block other queries, except very exclusive operations like `DROP TABLE` or `ALTER TABLE`.
- **granted (true):** This means the lock is currently held, not just requested.

This output is completely normal. When you run a `SELECT` on `pg_locks`, PostgreSQL automatically acquires an `AccessShareLock` on the table.
- This type of lock is harmless.
- It does not block other queries, except those requiring exclusive access, such as `DROP TABLE` or `ALTER TABLE`.
- Essentially, `pg_locks` is showing the lock that your own query implicitly holds on the system catalog.



---

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)