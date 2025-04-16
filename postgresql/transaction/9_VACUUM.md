# 🧹 VACUUM

```info
Author      Ter-Petrosyan Hakob
```
---

PostgreSQL uses [MVCC](./3_Multi_Version_Concurrency_Control.md){:target="_blank" rel="noopener"} to store multiple versions of the same data (called tuples) so different transactions can see the version they need. However, keeping these extra versions takes up more space on disk. Over time, if nothing is done, this unused data can fill your storage. To solve this, PostgreSQL includes a tool called **VACUUM**. Its job is to look at old tuple versions and remove the ones that are no longer needed.

> 🧠 **REMINDER:**
> A tuple is no longer visible (or perceivable) if no active transaction can still access it. 
> These outdated versions are called dead tuples, and they are safe to remove.

Running **VACUUM** helps clean up space, but it can also use a lot of `I/O` resources because it scans and updates storage. 
That’s why you don’t need to run it manually all the time. PostgreSQL includes a background process called **autovacuum**, which runs automatically based on how much activity your database has. This helps keep the system clean without interrupting your work.

The next sections will explain how to use both manual and automatic **VACUUM**.

## Manual VACUUM

You can run **VACUUM** manually on a single table, a set of table columns, or even an entire database. Here's the basic format:

```sql
VACUUM [ FULL ] [ FREEZE ] [ VERBOSE ] [ ANALYZE ] [ table_and_columns [, ...] ];
```

There are three main types of manual **VACUUM** operations, each one more aggressive than the last:

1) **Plain VACUUM:** This is the default. It removes dead tuples (old data no longer needed) but doesn't 
    shrink the table file on disk. So, while it helps keep things clean, it doesn’t actually free up storage space.

2) **VACUUM FULL:** This one rewrites the entire table, getting rid of both dead tuples and empty space. It reclaims real disk space 
    but is slower and more resource-intensive.

3) **VACUUM FREEZE: ** This version freezes old rows that will never change again. This prevents a serious issue known as 
    XID wraparound, which can corrupt your database over time if not managed.

> ⚠️ **NOTE:** You cannot run **VACUUM** inside a transaction, function, or stored procedure.

You can also add:

- **VERBOSE** to show detailed output during the process
- **ANALYZE** to update table statistics, which helps PostgreSQL make better decisions when running queries

### Trying Manual VACUUM

To test **VACUUM**, first make sure **autovacuum** is turned off, so it doesn’t interfere.

Run:

```sql
SHOW autovacuum;
```

If it returns:

```sql
autovacuum  
------------  
off  
(1 row)
```

You’re ready to proceed. If not, edit your **postgresql.conf** file (found in your **$PGDATA** directory), 
set **autovacuum = off**, then restart your PostgreSQL server.

Let’s take a closer look at the categories table in our PostgreSQL database.

- **Step 1:** Check the data

    ```sql
    SELECT * FROM categories;

    id |       name       
    ----+------------------
    1 | java
    2 | aws
    3 | javaScript
    6 | Eclipse IDE
    9 | IntelliJIdea IDE
    11 | Emacs Editor
    12 | Vi Editor
    13 | Atom Editor

    ```

- **Step 2:** Run `VACUUM ANALYZE`

    ```sql
    VACUUM ANALYZE categories;
    ```

    This command:

    - Cleans up the table (removes dead rows)
    - Updates statistics so PostgreSQL knows how many rows and pages the table uses

- **Step 3:** Check table size and metadata

    ```sql
    SELECT 
        relname, 
        reltuples, 
        relpages, 
        pg_size_pretty(pg_relation_size('categories'))
    FROM pg_class WHERE relname = 'categories' AND relkind = 'r';

    relname    | reltuples | relpages | pg_size_pretty 
    -----------+-----------+----------+----------------
    categories |        8  |        1 | 8192 bytes
    ```

    This data was selected from **pg_class**, a system catalog in PostgreSQL. It keeps track of tables, 
    indexes, sequences, and many other database objects.

    The condition `WHERE relkind = 'r'` means, only include regular tables (i.e., user-defined or system tables). 
    It excludes other object types like:
    - 'i' → indexes
    - 'S' → sequences
    - 'v' → views
    - 'm' → materialized views
    - 'c' → composite types
    - 'f' → foreign tables

    So yes—`relkind = 'r'` filters the query to only return real physical tables.


    📊 **What Do These Columns Mean?**

    | Column               | Description                                                                |
    |----------------------|----------------------------------------------------------------------------|
    | `relname`            | The name of the table (`categories`)                                       |
    | `reltuples`          | An estimated row count — here, it's 8, which matches the actual data       |
    | `relpages`           | The number of disk pages (blocks) used — here, 1 page = 8 KB               |
    | `pg_size_pretty(...)`| The formatted size of the table — 8192 bytes = 8 KB                        |


    🧠 **This kind of query helps you:**

    - 📏 Check table size before and after running VACUUM
    - 📦 Monitor storage usage
    - 🔍 Understand how PostgreSQL stores your data internally
    - 💡 Tip: You can run similar queries for other tables to keep track of space and performance in your database.

As you can see, the table currently has only 8 tuples and takes up just one data page on disk—about 8 KB in size. 
Now, let’s populate the table with around 1 million random tuples to observe how its size and structure change.

```sql
INSERT INTO categories( name ) SELECT 'GENERATED-#' || x FROM generate_series( 1, 1000000 ) x;
```

---


## 📌 Explore More

- 🏠 [Home](./../../README.md)
- 📚 [PostgreSql Tutorials](./../tutorials.md)
- 📜 [Write-Ahead Logging (WAL)](./8_Write_Ahead_Logging.md)