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

1) **Plain VACUUM**

    This is the default. It removes dead tuples (old data no longer needed) but doesn't shrink the table file on disk. 
    So, while it helps keep things clean, it doesn’t actually free up storage space.

2) **VACUUM FULL**

    This one rewrites the entire table, getting rid of both dead tuples and empty space. It reclaims real disk space but is slower and more resource-intensive.

3) **VACUUM FREEZE**

    This version freezes old rows that will never change again. This prevents a serious issue known as XID wraparound, which can corrupt 
    your database over time if not managed.

> ⚠️ **NOTE:** You cannot run **VACUUM** inside a transaction, function, or stored procedure.

---

## 📌 Explore More

- 🏠 [Home](./../../README.md)
- 📚 [PostgreSql Tutorials](./../tutorials.md)
- 📜 [Write-Ahead Logging (WAL)](./8_Write_Ahead_Logging.md)