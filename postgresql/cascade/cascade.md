# Understanding CASCADE in PostgreSQL

```info
Author      Ter-Petrosyan Hakob
Created     2025-04-08
Updated     2025-04-08
````

- [Foreign Key Constraints with CASCADE](#foreign-key-constraints-with-cascade)
    - [ON DELETE CASCADE](#on-delete-cascade)

---

In PostgreSQL, the **CASCADE** option is a powerful tool for managing dependencies between tables and other database objects. 
It ensures data integrity by automatically propagating changes—such as deletions or updates—across related rows and objects. 
In this post, we’ll explore how **CASCADE** works in the context of foreign key constraints and **DROP** commands, review 
alternatives, and discuss best practices.

## Foreign Key Constraints with CASCADE

When establishing relationships between tables, **CASCADE** provides automatic handling of dependent rows. 
This feature is available in both deletion and update scenarios:

### ON DELETE CASCADE

Using **ON DELETE CASCADE** in a foreign key constraint means that when a row in the parent table is deleted, 
all rows in the child table that reference that parent are automatically deleted. This feature helps maintain 
referential integrity by ensuring that no orphaned rows are left behind.

**Example:**
```sql
CREATE TABLE parent (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE child (
    id SERIAL PRIMARY KEY,
    parent_id INTEGER REFERENCES parent(id) ON DELETE CASCADE,
    description VARCHAR(100)
);
```

In this example, deleting a row from the `parent` table automatically deletes all corresponding rows in the `child` table.

---

[Home](./../../README.md) | [<< PostgreSql Tutorials](./../tutorials.md)