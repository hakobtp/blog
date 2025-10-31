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



---

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)
- [Understanding Locks: A Simple Introduction](./1_Understanding_Locks_A_Simple_Introduction.md)