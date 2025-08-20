# Introducing Transactions


```info
Author      Ter-Petrosyan Hakob
```

---

**Transactions** let a database group several operations together and treat them as one single unit, called an **atomic operation**. 
This means that either **all the changes happen**, or **none at all**—ensuring consistency.

PostgreSQL has a powerful and standards-compliant transaction system. It allows users to define specific **transaction properties**, including support for **nested transactions** using savepoints.

PostgreSQL relies on transactions to keep data **consistent and safe**, even when many users or processes are working at the same time. It uses a system called [Write-Ahead Logging (WAL)](./8_Write_Ahead_Logging.md) to protect data and make recovery possible if something goes wrong.

To support high performance with many simultaneous transactions, PostgreSQL also uses [Multi-Version Concurrency Control (MVCC)](./3_Multi_Version_Concurrency_Control.md). This allows multiple transactions to run at the same time without blocking each other.

A **transaction** is a group of actions that are treated as **one single unit**. It either completes fully or fails completely—there is no in-between.

Transactions are a core feature of any modern database system. They help databases follow the four important [ACID](./0_Understanding_ACID_in_PostgreSQL.md) properties:

- **Atomicity** – All steps in the transaction are done or none at all.
- **Consistency** – The database always moves from one valid state to another.
- **Isolation** – Transactions run as if they were the only ones happening.
- **Durability** – Once a transaction is committed, the changes are saved permanently.

Together, these properties make sure that data stays correct, safe, and reliable, even when multiple users or 
systems interact with the database at the same time.

You can think of a transaction as a group of related statements that will either all succeed together or all fail. Transactions are used everywhere in a database—even if you don’t notice them.

In fact, even simple actions like calling a function or running a single SQL statement are automatically wrapped in a tiny transaction. This means that every operation you run against the database is executed inside a transaction, even if you didn’t start one manually.

Thanks to this automatic behavior, the database can always keep your data safe and consistent, and prevent corruption. 

Sometimes, you may not want the database to automatically manage your statements. Instead, you might want to control when a transaction starts and ends. PostgreSQL allows this, and that’s where the concepts of **implicit** and **explicit** transactions come in.

- An **implicit** transaction is one the database starts for you automatically—like when you run a single SQL statement.
- An **explicit** transaction is one that you start manually using **BEGIN** and end with **COMMIT** or **ROLLBACK**.

Before we look at how these two types of transactions work and compare them, let’s review two important concepts:

- **Transaction ID (xid)**
    - Every transaction—whether implicit or explicit—is given a unique number called a transaction ID, or **xid**. 
        PostgreSQL automatically assigns an **xid** to each new transaction and guarantees that no two transactions 
        will share the same **xid** at the same time.
- **Tuples and xids**
    - PostgreSQL also stores the **xid** of the transaction that created or modified a row (called a tuple) inside the tuple itself.        

You’ll understand why this is important when we later explore how PostgreSQL handles multiple transactions at the 
same time—a concept called concurrency. For now, just remember: every row in every table is tagged with the xid (transaction ID) of the transaction that created or changed it.

PostgreSQL provides a special function called **txid_current()** that lets you check the transaction ID of the current transaction.

For example, try running a few simple queries like this:

```sql
SELECT current_time, txid_current();

   current_time    | txid_current 
-------------------+--------------
 11:24:22.18456+00 |         1888 


SELECT current_time, txid_current();

    current_time    | txid_current 
--------------------+--------------
 11:24:51.887052+00 |         1889
```

As you can see from the example above, the system assigned two different transaction IDs—1888 and 1889—to each **SELECT** statement. 
This confirms that each statement was executed in a separate implicit transaction.

**You may see different numbers on your own system, as transaction IDs increase over time.**

PostgreSQL stores the transaction ID that created each row in a special hidden column called **xmin**. 
You can query this column to find out which transaction inserted each row:

```sql
SELECT xmin, * FROM categories;
```

Example output:

```sql
 xmin | id | name 
------+----+------
  871 |  1 | java
  871 |  2 | c#
  872 |  3 | Rust
(3 rows)
```

In this example:
- Rows with id = 1 and id = 2 were created by transaction 871.
- The row with id = 3 was created by transaction 872.

PostgreSQL manages several of these hidden system columns that are not shown unless you explicitly request them. The most common ones include:

- **xmin** – the transaction ID that created the row
- **xmax** – the transaction ID that deleted the row (if applicable)
- **cmin** – the command ID of the inserting statement
- **cmax** – the command ID of the deleting or updating statement

## Implicit vs Explicit Transactions: What's the Difference?

**Implicit** transactions are transactions that you don’t manually start—PostgreSQL creates them for you automatically. In other words, PostgreSQL controls the transaction boundaries, deciding when a transaction begins and ends.

The rule is simple: each individual SQL statement runs in its own separate transaction.

To better understand this, let’s try inserting a few records into a table:

```sql
INSERT INTO categories( name ) VALUES( 'linux' );
INSERT INTO categories( name ) VALUES( 'Perl' );
INSERT INTO categories( name ) VALUES( 'javaScript' );

SELECT xmin, * FROM categories;

 xmin | id |    name    
------+----+------------
  871 |  1 | java
  871 |  2 | c#
  872 |  3 | Rust
  886 |  4 | linux
  887 |  5 | Perl
  888 |  6 | javaScript
(6 rows)
```

As you can see, the **xmin** field has a different (incrementing) value for each row that was inserted. 
This means that each **INSERT** statement was given a new transaction ID (**xid**).

In other words, every statement runs in its own transaction, even if you didn’t explicitly start one. Each **INSERT** 
is wrapped in a single-statement implicit transaction.

> **NOTE:**  The reason you see the **xid** values increasing by just one each time is because, 
> in these examples, no other database activity is happening. In other words, there’s no concurrency—no other users 
> or processes are running queries at the same time.
> 
> However, on a real or busy system with multiple connections and active transactions, you can’t predict what the next **xid** will be. 
> Other transactions may use up transaction IDs between your statements.

What if we wanted to insert all the previous categories at once, making sure that either all of them are saved, or none at all if something goes wrong?
To do this, we can use an explicit transaction.

An explicit transaction is a group of statements where you define the start and end of the transaction.

- You start the transaction with **BEGIN**.
- You end it with either **COMMIT** or **ROLLBACK**.
    - If you use **COMMIT**, all changes are saved permanently.
    - If you use **ROLLBACK**, the transaction is canceled, and all changes are undone.

Let’s see how this works by inserting a group of categories within a single explicit transaction:

```sql
BEGIN;

INSERT INTO categories( name ) VALUES( 'AWS' );
INSERT INTO categories( name ) VALUES( 'Kafka' );

COMMIT;

SELECT xmin, * FROM categories;

 xmin | id |    name    
------+----+------------
  871 |  1 | java
  871 |  2 | c#
  872 |  3 | Rust
  886 |  4 | linux
  887 |  5 | Perl
  888 |  6 | javaScript
  889 |  7 | AWS
  889 |  8 | Kafka
(8 rows)

```

Now, let’s see what happens if we end a transaction with **ROLLBACK** instead of **COMMIT**.
The expected result is that none of the changes should be saved.

As an example, let’s try to update all categories values to uppercase, but cancel the transaction afterward using **ROLLBACK**:

```sql
BEGIN;
UPDATE categories SET name = upper( name );
ROLLBACK;

SELECT name FROM categories;

name    
------------
 java
 c#
 Rust
 linux
 Perl
 javaScript
 AWS
 Kafka
(8 rows)

```
We first changed all the categories names to uppercase. But then we changed our mind and ran a **ROLLBACK**. 
At that moment, PostgreSQL discarded all the changes, and the data returned to its original state, as if nothing happened.

However, having control over a transaction doesn’t mean you can always decide how it ends. 
For example, if an error occurs during the transaction, PostgreSQL might force a rollback, and you won’t be allowed to **COMMIT** it—even if you want to.

The most basic example? A syntax error:

```sql
BEGIN;

UPDATE categories SET name = upperr( name );

ERROR:  function upperr(character varying) does not exist
LINE 1: UPDATE categories SET name = upperr( name );
                                     ^
HINT:  No function matches the given name and argument types. You might need to add explicit type casts.


COMMIT;
ROLLBACK
```

When PostgreSQL encounters an error, it automatically aborts the current transaction.

This means the transaction is still open, but it's marked as invalid. From that point on:
- PostgreSQL will ignore all your following commands.
- It will not allow you to **COMMIT** the transaction.
- As soon as you try to end the transaction, PostgreSQL will automatically perform a **ROLLBACK**.

In simple terms: after an error, the transaction is no longer usable. Even if you try to run more statements, PostgreSQL will refuse to accept them.

```sql
BEGIN;
INSERT INTO categories( name ) VALUES( 'AI' );

INSERT INTO categories( name ) VALUES( SomeText );
ERROR:  column "sometext" does not exist
LINE 1: INSERT INTO categories( name ) VALUES( SomeText );

INSERT INTO categories( name ) VALUES( 'design pattern' );
ERROR:  current transaction is aborted, commands ignored until end of transaction block

COMMIT;
ROLLBACK
```

Of course, syntax errors or misspelled table/column names aren't the only issues you might face when working with transactions. 
These are usually easy to fix.

However, sometimes your transaction might fail because of data-related issues, like a constraint violation. In such cases, the database prevents the transaction from continuing.

For example, imagine we add a rule that categories names must be at least 2 characters long. If we try to insert a categories that doesn’t meet this requirement, the transaction will fail.

Let’s see what happens when a constraint like this is triggered:

```sql
ALTER TABLE categories 
    ADD CONSTRAINT constraint_name_length CHECK ( length( name ) >= 2 );

BEGIN;

INSERT INTO categories( name ) VALUES( 'C' );

ERROR:  new row for relation "categories" violates check constraint "constraint_name_length"
DETAIL:  Failing row contains (10, C).

INSERT INTO categories( name ) VALUES( 'C++' );

ERROR:  current transaction is aborted, commands ignored until end of transaction block

COMMIT;
ROLLBACK

```

As you've seen, when a DML statement (like **INSERT**, **UPDATE**, or **DELETE**) fails, 
PostgreSQL immediately aborts the transaction. From that point on, the database refuses to process any more statements within that transaction.

The only way to fix the situation is to end the transaction, and it doesn’t matter whether you use **COMMIT** or **ROLLBACK**—PostgreSQL will roll back the changes anyway. This means everything done in the transaction will be discarded.

The same logic applies to implicit transactions:
If a single statement fails, PostgreSQL rolls back the transaction that wrapped that statement. As a result, none of the changes are saved.

In our earlier examples, we usually ended transactions with **COMMIT**, just to show how PostgreSQL refuses 
to commit bad or incomplete work. But in real-world scenarios, when you're not sure about the data, or an error occurs, 
you should use **ROLLBACK**. This cancels the transaction and keeps the database in a safe state.

Use an explicit transaction any time you have a group of operations that must all succeed or all fail together. 
This is especially important when partial success could lead to data inconsistency.

## Time Behavior Inside Transactions Explained

Transactions are time-discrete, which means that the time stays fixed during the entire transaction.
Even if you wait a few seconds and run **SELECT current_time**; again, you'll still get the same time—the one from when the transaction started.

If you need the actual, real-time clock while a transaction is running, you can use the function **clock_timestamp()**.
This will return the current system time, even inside a transaction.

Let’s see the difference in action with a quick example:

```sql
BEGIN;
SELECT CURRENT_TIME;

    current_time    
--------------------
 10:10:36.841265+00
(1 row)

SELECT pg_sleep_for( '5 seconds' );

SELECT CURRENT_TIME;

    current_time    
--------------------
 10:10:36.841265+00
(1 row)


SELECT CURRENT_TIME, clock_timestamp()::time;

    current_time    | clock_timestamp 
--------------------+-----------------
 10:10:36.841265+00 | 10:11:34.400918
(1 row)


SELECT pg_sleep_for( '5 seconds' );
SELECT CURRENT_TIME, clock_timestamp()::time;

    current_time    | clock_timestamp 
--------------------+-----------------
 10:10:36.841265+00 | 10:12:00.912546
(1 row)

```

---

## Explore More

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)
- [ACID](./0_Understanding_ACID_in_PostgreSQL.md) 
- [Transaction Identifiers](./2_transaction_identifiers.md)