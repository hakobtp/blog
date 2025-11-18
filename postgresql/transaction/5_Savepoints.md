# Savepoints

```info
Author      Ter-Petrosyan Hakob
```

---

A transaction is a block of work that must either succeed completely or fail completely. A savepoint allows you to break a 
transaction into smaller parts that can be rolled back individually, without canceling the entire transaction.

This is useful when you want a large transaction (with many statements) to continue, even if one part fails.

PostgreSQL does not support true nested transactions, so you can't use multiple **BEGIN** and **COMMIT/ROLLBACK** blocks inside one another.
However, savepoints let PostgreSQL mimic nested transactions by allowing you to mark rollback points within a transaction.

**How Savepoints Work**
- A savepoint is defined using a unique name.
- You can **ROLLBACK TO SAVEPOINT** to undo everything after that point.
- If you reuse the same savepoint name, the previous one is overwritten.


**Example: Rolling Back a Savepoint**

```sql
BEGIN;

INSERT INTO categories( name ) VALUES ( 'Eclipse IDE' );

SAVEPOINT other_categories;

INSERT INTO categories( name ) VALUES ( 'Netbeans IDE' );
INSERT INTO categories( name ) VALUES ( 'PhpStorm IDE' );

SELECT * FROM categories;

 id |     name     
----+--------------
  1 | java
  2 | aws
  3 | javaScript
  6 | Eclipse IDE
  7 | Netbeans IDE
  8 | PhpStorm IDE
(6 rows)


ROLLBACK TO SAVEPOINT other_categories;

INSERT INTO categories( name ) VALUES ( 'IntelliJIdea IDE' );

COMMIT;


SELECT * FROM categories WHERE name like '%IDE'; 

 id |       name       
----+------------------
  6 | Eclipse IDE
  9 | IntelliJIdea IDE
(2 rows)

```

In this example:
- Eclipse IDE was inserted before the savepoint and remains after the transaction is committed.
- Netbeans IDE and PhpStorm IDE were inserted after the savepoint and were rolled back.
- IntelliJIdea IDE was inserted after the rollback and committed successfully.


**Releasing Savepoints**

You can also choose to release a savepoint using **RELEASE SAVEPOINT**, which means:
- The savepoint is removed.
- The statements before and after it now follow the main transaction.

**Example:**

```sql
BEGIN;

SAVEPOINT editors;

INSERT INTO categories( name ) VALUES ( 'Emacs Editor' );
INSERT INTO categories( name ) VALUES ( 'Vi Editor' );

RELEASE SAVEPOINT editors;
INSERT INTO categories( name ) VALUES ( 'Atom Editor' );

COMMIT;

SELECT * FROM categories WHERE name like '%Editor'; 
 id |     name     
----+--------------
 11 | Emacs Editor
 12 | Vi Editor
 13 | Atom Editor
(3 rows)

```

After **RELEASE SAVEPOINT**, the earlier inserts are no longer grouped—they behave as if they were part of the main transaction.


**Rolling Back with Multiple Savepoints**

If you roll back to a savepoint, PostgreSQL removes that savepoint and all that came after it.

Example:

```sql
BEGIN;
SAVEPOINT perl;
INSERT INTO categories( name ) VALUES ( 'Rakudo Compiler' );

SAVEPOINT gcc;
INSERT INTO categories( name ) VALUES ( 'Gnu C Compiler' );


ROLLBACK TO SAVEPOINT perl;
COMMIT;

SELECT name FROM categories WHERE name LIKE '%Compiler';
 name 
------
(0 rows)

```

Here, even though the transaction was committed, everything after the `perl` savepoint was undone, including `gcc`.

- **In short:**
    - Rolling back to a savepoint undoes all statements after that point. Anything before it stays safe—until the transaction ends.

- **Coming Up:**
    - Transactions can sometimes lead to a situation where the database can't move forward—this is called a deadlock.

---

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)
- [Non-Repeatable vs Phantom Reads: What's the Real Difference?](./4_Non_Repeatable_vs_Phantom_Reads.md)
- [Isolation levels](./6_Isolation_levels.md)