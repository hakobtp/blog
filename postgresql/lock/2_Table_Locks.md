# Table Locks

A table lock doesn’t necessarily block all activity on the table. PostgreSQL supports eight table-level lock types, 
and a transaction can hold multiple locks on the same table. Some locks allow other operations to continue, while others block almost everything.

To check which locks are held on your tables, you can query the `pg_locks` system view:

```sql
SELECT pid, relation::regclass, mode, granted
FROM pg_locks
WHERE relation IS NOT NULL;
```

Example output:

```bash
24088 | pg_locks | AccessShareLock | true
```

Explanation:

- **pid (24088):** The session ID holding the lock.
- **relation::regclass (pg_locks):** The table that is locked.
- **mode (AccessShareLock):** The type of lock. AccessShareLock is the least restrictive, acquired automatically by SELECT queries.
- **granted (true):** The lock is currently held (not just requested).

This is normal. When you query `pg_locks`, PostgreSQL acquires an implicit `ACCESS SHARE` lock on the system catalog.

To identify queries blocked by other sessions, use:
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

This query shows which sessions are blocked and which are blocking, helping you troubleshoot lock contention.

## ACCESS EXCLUSIVE — The Strongest Lock

`ACCESS EXCLUSIVE` is the most restrictive lock. When a session holds it, **no one else can read or write the table**, and almost all commands are blocked.

Commands that acquire this lock include:

```sql
DROP TABLE
TRUNCATE
VACUUM FULL
REINDEX
CLUSTER
ALTER TABLE ...
REFRESH MATERIALIZED VIEW
```

**Example:**

```sql
-- Session 1
VACUUM FULL users;  -- Acquires ACCESS EXCLUSIVE lock
```

While this runs:

```sql
-- Session 2
SELECT * FROM users;  -- Blocks until Session 1 finishes

INSERT INTO users(name, email) VALUES ('Alice', 'alice@example.com');  -- Also blocked
```

Once `VACUUM FULL` completes, all blocked queries continue.

**Key takeaway:** Use `ACCESS EXCLUSIVE` for operations that require complete isolation, such as restructuring or rebuilding a table.

## ACCESS SHARE — The Lightest Lock

`ACCESS SHARE` is acquired by read-only commands, like `SELECT` or `COPY TO`. It rarely blocks other sessions.

```sql
SELECT * FROM users;
COPY users TO '/path/to/file.csv' WITH (FORMAT csv, HEADER);
```

**Behavior:**
- Conflicts only with `ACCESS EXCLUSIVE` operations.
- Multiple readers can access the table simultaneously.
- Writers (`INSERT`, `UPDATE`, `DELETE`) are generally allowed alongside readers.

**Example:**
```sql
-- Session 1
SELECT * FROM users;

-- Session 2
SELECT * FROM users;  -- Executes immediately

-- Session 3
INSERT INTO users(name, email) VALUES ('Charlie', 'charlie@example.com');  -- Allowed

-- Session 4
VACUUM FULL users;  -- Waits until readers finish
```


## Table Lock Matrix (Simplified)

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

Stronger locks block more operations. The further down the list, the more restrictive the lock.

## Conclusion

PostgreSQL’s locking system ensures data consistency while allowing high concurrency. Understanding which locks your queries acquire helps you write safer, faster, and more predictable SQL. By knowing when and how locks block operations, you can avoid unnecessary conflicts and optimize performance.

---

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)
- [Understanding Locks: A Simple Introduction](./1_Understanding_Locks_A_Simple_Introduction.md)