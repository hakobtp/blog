# Multi-Version Concurrency Control

```info
Author      Ter-Petrosyan Hakob
```

---

What happens if two processes try to change the same data at the same time? PostgreSQL must keep the data correct, so it uses 
locks to block and protect data from conflicting changes. However, locks can make the system slower because many locks can force transactions to wait.

To reduce this problem, PostgreSQL uses something called Multi-Version Concurrency Control (**MVCC**). With **MVCC**, 
the system does not change a record directly. Instead, it makes a copy of the record, applies the changes to the copy, and marks the original record as outdated. This idea is similar to the "copy-on-write" method used in filesystems like ZFS.

For example, imagine a table called `categories` that has three records. If you update the description of one record, PostgreSQL creates a new record with the changes and invalidates the old one.

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
          871 | 871:871:
(1 row)

testdb=*# UPDATE categories SET name = lower(name);
UPDATE 2


```

In the example above, the transaction number is 871, and its snapshot includes only itself (871:871:). This means the transaction sees only data already saved and visible in the database before transaction 871 started.

Now, let's pause this session for a moment. Open another session connected to the same database. In this new session, 
perform a single **INSERT** statement. This statement runs inside its own implicit transaction. Then, retrieve the transaction’s **xid** to see its identifier.

```sql
-- SESSION 2

testdb=# INSERT INTO categories( name ) VALUES( 'Rust' ) RETURNING txid_current();

 txid_current 
--------------
          872
(1 row)

INSERT 0 1

```

The single-action transaction got the **xid** number 872. This number is the next available **xid** after the previous transaction (871). 
We didn't run any other transactions at the same time, which makes it easier to follow the numbers clearly.

Now, let's return to the first session and check the snapshot information again:

```sql
- SESSION 1

testdb=*#  SELECT txid_current(), txid_current_snapshot();

 txid_current | txid_current_snapshot 
--------------+-----------------------
          871 | 871:873:
(1 row)

```

This time, the snapshot of our first transaction has grown. Its range now includes transaction 873, which hasn't started yet 
(the function **txid_current_snapshot()** shows the upper limit as non-inclusive).

In other words, the first transaction can now see data that was added by transaction 872, even though 872 started later. This becomes even clearer if we query the table within the first transaction:


```sql
- SESSION 1

testdb=*# SELECT xmin, name FROM categories;

 xmin | name 
------+------
  871 | java
  871 | c#
  872 | Rust
(3 rows)

```

As you can see, all the records except the last one were created by the current transaction. The last record was created by transaction 872.

But the previous example shows only part of the story. While the first transaction is still open (not completed), let's check the same table again from another parallel session:

```sql
- SESSION 2

testdb=# SELECT xmin, name FROM categories;

  xmin | name 
------+------
  869 | Java
  870 | C#
  872 | Rust
(3 rows)

```

All records except the last one have different descriptions and, importantly, different **xmin** values compared to what transaction 871 sees. 
What does this mean?

It means that even though most of the table was updated (all records except the last one), other transactions running at the same time can still see the data without waiting for locks.

This is the main idea of MVCC: each transaction sees its own version of the data. The exact data a transaction sees depends on the snapshot (the specific time range) associated with that transaction.

Sooner or later, the changes made to the data must be finalized. When transaction 871 completes (by running **COMMIT**), 
its changes become permanent. After that, every new transaction will see this updated data as the current and true state of the database.

```sql
-- SESSION 1

testdb=*# COMMIT;
COMMIT

-- The transaction is now finished.
-- Everyone sees the final updated data.

testdb=# SELECT xmin, name FROM categories;

 xmin | name 
------+------
  871 | java
  871 | c#
  872 | Rust
(3 rows)
```


```sql
-- SESSION 2

testdb=# SELECT xmin, name FROM categories;
 xmin | name 
------+------
  871 | java
  871 | c#
  872 | Rust
(3 rows)

```

MVCC does not always avoid the use of locks. If two or more transactions try to change the same data at the same time, PostgreSQL must put these transactions in order. To do this, the database must use locks, allowing only one transaction to continue while the others wait.

We can easily demonstrate this behavior using two parallel sessions, just like in the previous example:

```sql
-- SESSION 1

testdb=# BEGIN;

testdb=*# SELECT txid_current(), txid_current_snapshot();

 txid_current | txid_current_snapshot 
--------------+-----------------------
          873 | 873:873:
(1 row)


testdb=*# UPDATE categories SET name = upper( name );
UPDATE 3
```

Meanwhile, run the following commands in another session:

```sql
-- SESSION 2
testdb=# BEGIN;

testdb=*# SELECT txid_current(), txid_current_snapshot();

 txid_current | txid_current_snapshot 
--------------+-----------------------
          874 | 873:873:
(1 row)


testdb=*# UPDATE categories SET name = lower( name );

-- LOCKED!!!!
```

Transaction 874 is locked because PostgreSQL doesn't know which update should be applied first. 
Transaction 873 is changing all names to uppercase, and at the same time, transaction 874 is changing the same names to lowercase.

Since these two transactions conflict, PostgreSQL must decide the order clearly. The final result depends on which transaction finishes last. 
Because transaction 873 started changing the data first, transaction 874 is blocked. It must wait for transaction 873 to finish (**COMMIT** or **ROLLBACK**).

When you finish the first transaction (873), the second transaction (874) will immediately continue, and you will see the result of the previously blocked **UPDATE** statement.

```sql
-- session 1
testdb=*# COMMIT;
COMMIT

-- session 2
UPDATE 3
-- The transaction is unblocked and can continue.
testdb=*#
```

As we've seen, MVCC doesn't completely prevent locking. However, it significantly reduces locking and improves the overall performance by allowing multiple transactions to run concurrently (at the same time).

MVCC does have one disadvantage: the database must store several versions of each record because different transactions may see different versions of the data. This means the database can grow larger than just the consolidated (final) data.

To solve this issue, PostgreSQL has special tools called **VACUUM** and **autovacuum**. These tools scan tables and indexes to find old versions of records that are no longer needed and remove them to free up storage space.

When does **VACUUM** delete an old record version? It deletes it when no active transactions can see it anymore—that is, when no transactions still refer to its **xmin** (its creation transaction ID). In other words, a record version can be removed once it’s no longer visible to any ongoing transactions.

## Detailed Explanation of Internal Fields

**xmin** is just one part of how PostgreSQL manages **MVCC**.

PostgreSQL labels every row (tuple) in the database with four internal system fields: **xmin**, **xmax**, **cmin**, and **cmax**.

Just like with **xmin**, you need to explicitly select these fields if you want to see them in your queries.

```sql
SELECT xmin, xmax, cmin, cmax, * FROM categories ORDER BY name;

 xmin | xmax | cmin | cmax | id |    name    
------+-------+------+-------+----+------------
  889 |  890 |    0 |    0 |  7 | AWS
  871 |  890 |    0 |    0 |  2 | c#
  871 |  890 |    0 |    0 |  1 | java
  888 |  890 |    0 |    0 |  6 | javaScript
  889 |  890 |    0 |    0 |  8 | Kafka
  886 |  890 |    0 |    0 |  4 | linux
  887 |  890 |    0 |    0 |  5 | Perl
  872 |  890 |    0 |    0 |  3 | Rust
(8 rows)
```

**What Each Field Means:**

- **xmin** – The transaction ID (xid) of the transaction that created the row.
- **xmax** – The xid of the transaction that deleted or invalidated the row (if any).
- **cmin** – The command ID of the statement that created the row within the transaction.
- **cmax** – The command ID of the statement that deleted or updated the row.

> PostgreSQL assigns a unique command ID to each statement inside a transaction, starting from 0.

**Why Are cmin and cmax Important?**

PostgreSQL’s default isolation level is **READ_COMMITTED**.
This means each statement in a transaction must see the database as it was when that statement began.

To make this work properly, PostgreSQL needs to track:
- Which statement created each row
- Which statement modified or deleted it

That’s exactly what **cmin** and **cmax** are for—they allow PostgreSQL to decide what each 
statement can `"see"`, even inside the same transaction. You can see how **cmin** and **cmax** are used within the same 
transaction in the example below.

First, we start an explicit transaction using **BEGIN**.
Then, we run two separate **INSERT** statements, each adding a row to the table. Since each statement inside a transaction gets a unique command ID, the inserted rows will have different **cmin** values, even though they were created in the same transaction.

Let’s take a look:

```sql
BEGIN;

SELECT xmin, xmax, cmin, cmax, name, txid_current() FROM categories ORDER BY name;

xmin | xmax | cmin | cmax |    name    | txid_current 
------+------+------+------+------------+--------------
  903 |    0 |    0 |    0 | aws        |          905
  902 |    0 |    0 |    0 | java       |          905
  904 |    0 |    0 |    0 | javaScript |          905


-- first writing command (number 0)
INSERT INTO categories( name ) values( 'go' );

-- second writing command (number 1)
INSERT INTO categories( name ) values( 'Ruby' );

SELECT xmin, xmax, cmin, cmax, name, txid_current() FROM categories ORDER BY name;

 xmin | xmax | cmin | cmax |    name    | txid_current 
------+------+------+------+------------+--------------
  903 |    0 |    0 |    0 | aws        |          905
  905 |    0 |    0 |    0 | go         |          905
  902 |    0 |    0 |    0 | java       |          905
  904 |    0 |    0 |    0 | javaScript |          905
  905 |    0 |    1 |    1 | Ruby       |          905
(5 rows)

```

So far, in this transaction, we’ve inserted two new rows. If you look at the result of the second **SELECT** query, you’ll notice that:

- Both of the new rows (go and Ruby) have the same **xmin** value: 905, which matches 
  the output of **txid_current()**. This confirms that both rows were created by the same transaction.
- However, the **cmin** values are different:
  - `go` has **cmin = 0**
  - `Ruby` has **cmin = 1**

That’s because they were inserted by two different **INSERT** commands within the same transaction.
PostgreSQL starts counting commands from 0, so every statement inside a transaction is given a unique command ID.

> This shows that PostgreSQL not only tracks which transaction created a row, but also which specific command within that transaction.


Let’s continue the transaction by opening a cursor that selects from the categories table, and then delete all rows except two. The transaction session continues like this:

```sql
DECLARE category_cursor CURSOR FOR 
  SELECT xmin, xmax, cmin, cmax, name, txid_current() 
  FROM categories 
  ORDER BY name;

DELETE FROM categories WHERE name NOT IN ( 'go', 'javaScript' );

SELECT xmin, xmax, cmin, cmax, name, txid_current() FROM categories ORDER BY name;

 xmin | xmax | cmin | cmax |    name    | txid_current 
------+------+------+------+------------+--------------
  905 |    0 |    0 |    0 | go         |          905
  904 |    0 |    0 |    0 | javaScript |          905
(2 rows)


```
As you can see, the table now contains only two rows. This is the expected result after the **DELETE** statement.

However, there’s an important detail here:
The cursor was opened before the **DELETE** statement was executed.
Because of this, the cursor sees the snapshot of the data as it existed when the cursor was declared.

Let’s check what the cursor can still access: even though the rows were deleted from the table, the cursor can still return all the rows that existed before the deletion.

```sql
FETCH ALL FROM category_cursor;

 xmin | xmax | cmin | cmax |    name    | txid_current 
------+------+------+------+------------+--------------
  903 |  905 |    2 |    2 | aws        |          905
  905 |    0 |    0 |    0 | go         |          905
  902 |  905 |    2 |    2 | java       |          905
  904 |    0 |    0 |    0 | javaScript |          905
  905 |  905 |    0 |    0 | Ruby       |          905
(5 rows)

```

There’s an important detail to notice here:

Each deleted row now has an xmax value set to the current transaction ID (905).
This tells us that this transaction is responsible for deleting those rows.

However, because the transaction hasn’t been committed yet, the deleted rows are still physically present in the table.
They’re just marked as invisible for any snapshots that end at transaction 905 or later.

Also, these deleted rows have **cmax = 2**, which means they were removed during the third writing command inside the transaction (remember, command numbering starts from 0).

Since the cursor was declared before the **DELETE** statement, it continues to see the data as it existed at the time the cursor was created—even though PostgreSQL already knows that those rows are marked for deletion.

You may also notice that **cmin** and **cmax** sometimes have the same value.
This is because both fields share the same internal storage, and PostgreSQL uses one or the other depending on whether the row was inserted, updated, or deleted.

## Summary

**MVCC** stands for **Multi-Version Concurrency Control**. It's an important feature in PostgreSQL and many other modern databases. 
**MVCC** allows multiple transactions to read and write data at the same time, ensuring the data remains consistent and correct.

### How does MVCC work?

1) **Versions of Data**
  - When you insert or update data, PostgreSQL doesn't immediately overwrite the existing data.
  - Instead, it creates a new version of the data (called a tuple) with a new transaction ID (**xid**).
  - The old data version is kept but marked as outdated.

2) **Visibility (Snapshots)**
  - Every transaction sees data based on its snapshot, created at the transaction’s start.
  - A transaction sees only data versions committed (saved permanently) before it began.
  - This gives each transaction a consistent view of data, even if other transactions are changing data at the same time.

3) **Better Concurrency**
  - **MVCC** allows multiple transactions to read the same data simultaneously without waiting or blocking each other.
  - Transactions that write (change data) still need row-level locks to prevent conflicts.

4) **Transaction Isolation Levels**
  - **MVCC** helps PostgreSQL support different isolation levels (Read Committed, Repeatable Read, Serializable).
  - Each isolation level decides exactly which data a transaction can see, based on **MVCC** snapshots.

5) **Cleaning Up Old Data (Vacuuming)**
  - PostgreSQL regularly runs a process called vacuuming.
  - Vacuuming removes old versions of data that are no longer visible or needed.
  - This prevents the database from growing unnecessarily large over time.

---

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)
- [Transaction Identifiers](./2_transaction_identifiers.md)
- [Non-Repeatable vs Phantom Reads: What's the Real Difference?](./4_Non_Repeatable_vs_Phantom_Reads.md)