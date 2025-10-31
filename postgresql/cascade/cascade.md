# Understanding CASCADE in PostgresSQL

```info
Author      Ter-Petrosyan Hakob
Created     2025-04-08
Updated     2025-04-08
```

- [Foreign Key Constraints with CASCADE](#foreign-key-constraints-with-cascade)
    - [ON DELETE CASCADE](#on-delete-cascade)
    - [ON UPDATE CASCADE](#on-update-cascade)
    - [Alternatives: ON DELETE SET NULL and ON DELETE SET DEFAULT](#alternatives-on-delete-set-null-and-on-delete-set-default)
- [CASCADE with DROP Commands](#cascade-with-drop-commands)
- [Summary of Cascade Options in PostgreSQL](#summary-of-cascade-options-in-postgresql)
- [Best Practices for Using CASCADE](#best-practices-for-using-cascade)

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

### ON UPDATE CASCADE

Similarly, **ON UPDATE CASCADE** ensures that if the primary key of a row in the parent table is updated, the corresponding foreign key values in the child table are updated accordingly. This keeps related data consistent when primary keys change.

**Example:**
```sql
CREATE TABLE parent (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE child (
    id SERIAL PRIMARY KEY,
    parent_id INTEGER REFERENCES parent(id) ON UPDATE CASCADE,
    description VARCHAR(100)
);
```

Here, any update to the `id` in `parent` will automatically be reflected in the `child` table’s `parent_id` column.

### Alternatives: ON DELETE SET NULL and ON DELETE SET DEFAULT

Not every situation calls for a full cascade delete. Two common alternatives are:

- **ON DELETE SET NULL:** When the `parent` row is deleted, the foreign key in the `child` table is set to **NULL**. 
    This is ideal for optional relationships where the `child` row can exist without a `parent`.
    ```sql
        CREATE TABLE child (
            id SERIAL PRIMARY KEY,
            parent_id INTEGER REFERENCES parent(id) ON DELETE SET NULL,
            description VARCHAR(100)
        );
    ```
- **ON DELETE SET DEFAULT:** In this case, deleting a `parent` row sets the foreign key in the 
    `child` table to its default value. This option requires that the foreign key column has an appropriate default value defined.   
    ```sql
        CREATE TABLE child (
            id SERIAL PRIMARY KEY,
            parent_id INTEGER DEFAULT 1 REFERENCES parent(id) ON DELETE SET DEFAULT,
            description VARCHAR(100)
        );
    ``` 

## CASCADE with DROP Commands

**CASCADE** can also be used with **DROP** commands to simplify the deletion of database objects by automatically 
removing any dependent objects. This can be very useful—but also dangerous if not managed carefully.

**Examples:**
- **DROP TABLE ... CASCADE:** Drops the table along with any objects (such as foreign key references) that depend on it.
    ```sql
    DROP TABLE parent CASCADE;
    ```
- **DROP VIEW ... CASCADE:** Drops a view and any views that depend on it.
    ```sql
    DROP VIEW my_view CASCADE;
    ```
- **DROP FUNCTION ... CASCADE:** Drops a function and any database objects (views, triggers, etc.) that depend on it.
    ```sql 
    DROP FUNCTION my_function CASCADE;
    ```

    
## Summary of Cascade Options in PostgreSQL

|Usage	                    |Description                                                                                    |
|:--------------------------|:----------------------------------------------------------------------------------------------|
|**ON DELETE CASCADE**      |Deletes related rows in the child table when a row in the parent table is deleted.             |
|**ON UPDATE CASCADE**      |Updates foreign key values in the child table when a primary key in the parent table changes.  |
|**ON DELETE SET NULL**     |Sets the foreign key in the child table to NULL when the referenced parent row is deleted.     |
|**ON DELETE SET DEFAULT**  |Sets the foreign key in the child table to its default value upon deletion of the parent row.  |
|**DROP ... CASCADE**       |Drops the target object along with all dependent objects (e.g., foreign keys, views, triggers).|


## Best Practices for Using CASCADE

- **Use with Caution:** Cascading operations, particularly **ON DELETE CASCADE**, may remove more data than anticipated 
    if not implemented carefully. Always verify your relationships and understand the chain of dependencies.
- **Assess Referential Integrity Requirements:** Decide whether child records should be deleted, updated, set to **NULL**, 
    or handled in another specific way when the parent record changes.
- **Plan Object Dependencies:** When using **DROP ... CASCADE**, double-check dependencies to avoid accidentally deleting critical database objects.

By thoughtfully applying these principles, you can leverage **CASCADE** to simplify database maintenance while keeping your data safe and consistent.

---

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)