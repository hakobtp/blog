# Multi-Version Concurrency Control

```info
Author      Ter-Petrosyan Hakob
Created     2025-04-09
Updated     2025-04-09
```

---

What happens if two processes try to change the same data at the same time? PostgreSQL must keep the data correct, so it uses 
locks to block and protect data from conflicting changes. However, locks can make the system slower because many locks can force transactions to wait.

To reduce this problem, PostgreSQL uses something called Multi-Version Concurrency Control (**MVCC**). With **MVCC**, 
the system does not change a record directly. Instead, it makes a copy of the record, applies the changes to the copy, and marks the original record as outdated. This idea is similar to the "copy-on-write" method used in filesystems like ZFS.

For example, imagine a table called "categories" that has three records. If you update the description of one record, PostgreSQL creates a new record with the changes and invalidates the old one.

Why does PostgreSQL do extra work with **MVCC** instead of updating data directly? This extra work helps the database keep multiple versions of the same data at the same time. Each version of the data is correct during a specific time period. Because of this, PostgreSQL does not need many locks, and different transactions can see different versions of the data without waiting.

To make **MVCC** work, PostgreSQL uses something called **snapshots**. A snapshot defines exactly which versions of the data a transaction can see. 
Each snapshot has a range of **xids** [transaction identifiers](./2_transaction_identifiers.md). If a row's **xid** is inside the snapshot's range, the current transaction can see and use that row. Simply put, every transaction sees only a part of all data in the database. Checking exactly which rows a transaction can see is more complicated, but the main idea is as described.

PostgreSQL has a special function called **txid_current_snapshot()**. This function returns the lowest and highest **xids** 
that define the range of data visible to the current transaction. We can easily show how this works using two parallel sessions.

In the first session, we start a transaction, get its **xid** and snapshot to use later, and then perform an action.

```sql
-- SESSION 1

testdb=# BEGIN;

testdb=*# SELECT txid_current(), txid_current_snapshot();

 txid_current | txid_current_snapshot 
--------------+-----------------------
          858 | 858:858:
(1 row)

testdb=*# UPDATE categories SET name = lower(name);
UPDATE 2


```

In the example above, the transaction number is 858, and its snapshot includes only itself (858:858:). This means the transaction sees only data already saved and visible in the database before transaction 858 started.

Now, let's pause this session for a moment. Open another session connected to the same database. In this new session, 
perform a single **INSERT** statement. This statement runs inside its own implicit transaction. Then, retrieve the transaction’s **xid** to see its identifier.

```sql
-- SESSION 2

testdb=# INSERT INTO categories( name ) VALUES( 'Rust' ) RETURNING txid_current();

 txid_current 
--------------
          859
(1 row)

INSERT 0 1

```

The single-action transaction got the **xid** number 859. This number is the next available **xid** after the previous transaction (858). 
We didn't run any other transactions at the same time, which makes it easier to follow the numbers clearly.

Now, let's return to the first session and check the snapshot information again:

```sql
- SESSION 1

testdb=*#  SELECT txid_current(), txid_current_snapshot();

 txid_current | txid_current_snapshot 
--------------+-----------------------
          858 | 858:860:
(1 row)

```

This time, the snapshot of our first transaction has grown. Its range now includes transaction 860, which hasn't started yet 
(the function `txid_current_snapshot()` shows the upper limit as non-inclusive).

In other words, the first transaction can now see data that was added by transaction 859, even though 859 started later. This becomes even clearer if we query the table within the first transaction:


```sql
- SESSION 1

testdb=*# SELECT xmin, name FROM categories;

 xmin | name 
------+------
  858 | java
  858 | c#
  859 | Rust
(3 rows)

```

As you can see, all the records except the last one were created by the current transaction. The last record was created by transaction 859.

But the previous example shows only part of the story. While the first transaction is still open (not completed), let's check the same table again from another parallel session:

```sql
- SESSION 2

testdb=# SELECT xmin, name FROM categories;

 xmin | name 
------+------
  856 | Java
  856 | C#
  859 | Rust
(3 rows)

```

[Home](./../../README.md) | [PostgreSql Tutorials](./../tutorials.md) | [Transaction Identifiers](./2_transaction_identifiers.md)