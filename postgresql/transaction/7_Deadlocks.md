# Deadlocks

```info
Author      Ter-Petrosyan Hakob
```
---

A **deadlock** happens when **two or more transactions block each other in a circular way**, each waiting for the other to release a resource.

In a busy, multi-user database, deadlocks are actually normal from time to time.
They are not usually something a database administrator needs to worry about—**unless they start happening often**.

If deadlocks become frequent, it may indicate a design or logic issue in the application or how transactions are written.

When a deadlock occurs, the only solution is to terminate one of the transactions that are involved.

PostgreSQL includes a powerful deadlock detection system that handles this automatically.
It watches for transactions that get stuck, and when it detects a deadlock, it cancels one of the transactions and performs a **ROLLBACK**.

To understand how a deadlock can happen, imagine two transactions running at the same time, both trying to modify the same rows in a way that causes conflict.

For example, the first transaction might look like this:

```sql
-- session 1
BEGIN;

SELECT txid_current();

 txid_current 
--------------
         2099
(1 row)

UPDATE categories SET name = 'Perl 5' WHERE name = 'perl';
```

And in the meantime, the other transaction performs the following:

```sql
-- session 2
BEGIN;

SELECT txid_current();
 txid_current 
--------------
         2100
(1 row)

UPDATE categories SET name = 'Java and Groovy' WHERE name = 'java';
```

So far, both transactions have updated different rows, so they haven't caused any conflict.

But now, imagine the first transaction tries to modify a row that the second transaction has already updated. 
PostgreSQL will lock the row, and the first transaction will have to wait until it can get access.

At this point, the transaction becomes blocked, waiting to acquire a row-level lock.

```sql
-- session 1
UPDATE categories SET name = 'The Java Language' WHERE name = 'java';
-- locked
```

If the second transaction tries, on the other hand, to modify a tuple already touched by the first transaction, it will be locked, waiting for the lock acquisition:

```sql
-- session 2
UPDATE categories SET name = 'Perl and Raku' WHERE name = 'perl';

ERROR:  deadlock detected
DETAIL:  Process 3638 waits for ShareLock on transaction 2099; blocked by process 3556.
Process 3556 waits for ShareLock on transaction 2100; blocked by process 3638.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,12) in relation "categories"
```

This time, however, PostgreSQL detects a circular dependency:
Each transaction is waiting for the other to release a lock, and neither can continue.

When this happens, PostgreSQL realizes there’s a deadlock—and to resolve it, it automatically cancels one of the transactions.

In this example, PostgreSQL chooses to terminate the second transaction, so the first one can continue.

As shown in the error message, PostgreSQL clearly identifies that transaction 2099 is waiting for a lock held by transaction 2100, and transaction 2100 is also waiting on 2099.
Since there’s no way forward, PostgreSQL must cancel one to break the cycle.


## How PostgreSQL Detects Deadlocks

Deadlock detection is a complex and resource-heavy process, so PostgreSQL doesn’t run it constantly.
Instead, it checks for deadlocks on a schedule.

This behavior is controlled by the **deadlock_timeout** setting.

- It defines how long PostgreSQL should wait before checking for a deadlock.
- The value is expressed in milliseconds.
- By default, it’s set to 1000 milliseconds (1 second).

This delay helps PostgreSQL avoid doing expensive checks too often, while still resolving real deadlocks efficiently.


```sql
SELECT name, setting, unit FROM pg_settings WHERE name like '%deadlock%';

       name       | setting | unit 
------------------+---------+------
 deadlock_timeout | 1000    | ms
(1 row)

SHOW deadlock_timeout;

 deadlock_timeout 
------------------
 1s
(1 row)
```

You can temporarily change it in a session (for testing or fine-tuning):

```sql
SET deadlock_timeout = '500ms'; -- half a second

```

To make a permanent change, you’ll need to edit the `postgresql.conf` file:

```conf
# postgresql.conf
deadlock_timeout = '750ms'  -- tune based on how quickly your system detects blocking
```

After editing the file, restart or reload PostgreSQL:

```sql
# Reload config without restart
SELECT pg_reload_conf();
```

**Be careful when decreasing deadlock_timeout.**

Lowering this value means PostgreSQL will detect deadlocks sooner, and your transactions will **fail faster** if they're in a deadlock situation.

However, there's a downside:
PostgreSQL will spend more resources checking for dependencies, which can increase CPU usage and affect performance—especially in high-concurrency environments.

So, unless you have a specific reason, don’t set this value too low. It’s often better to leave it at the default or fine-tune it carefully based on testing.

---

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)
- [Isolation levels](./6_Isolation_levels.md)
- [Write-Ahead Logging (WAL)](./8_Write_Ahead_Logging.md)