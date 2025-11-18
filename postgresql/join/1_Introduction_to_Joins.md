# Introduction to Joins

```info
Author      Ter-Petrosyan Hakob
```
---

When you keep related data in two tables, you often need to bring it together. A **join** tells PostgreSQL how to match rows from one table with rows from another. In this post, In this post series, we will look at seven basic joint types:

| JOIN TYPE     | What It Does                                                                                                 |
|---------------|--------------------------------------------------------------------------------------------------------------|
| **INNER JOIN**   | Shows only rows that match in both tables.                                                                 |
| **LEFT JOIN**    | Shows all rows from the first table, plus matching rows from the second. Unmatched rows show `NULL`.       |
| **RIGHT JOIN**   | Shows all rows from the second table, plus matching rows from the first. Unmatched rows show `NULL`.       |
| **FULL JOIN**    | Shows all rows from both tables. If there is no match, one side shows `NULL`.                              |
| **CROSS JOIN**   | Pairs every row in the first table with every row in the second (Cartesian product).                       |
| **SELF JOIN**    | Joins a table to itself so you can compare rows within the same table.                                     |
| **LATERAL JOIN** | Runs a subquery for each row of the first table, letting the subquery use that row’s values.              |



## Setting Up Sample Tables

For the upcoming tutorials series, let’s create two sample tables:

1. **categories** – one row per category. Each category can have a parent category.  
2. **goods**      – one row per product, linked to a category.

```sql
CREATE TABLE categories (
  id        SERIAL PRIMARY KEY,
  name      VARCHAR(50) NOT NULL,
  parent_id INTEGER REFERENCES categories(id)
);

CREATE TABLE goods (
  id          SERIAL PRIMARY KEY,
  name        VARCHAR(100) NOT NULL,
  category_id INTEGER NOT NULL REFERENCES categories(id),
  price       NUMERIC(8,2) NOT NULL
);

```

Now let’s insert some sample data:

```sql
-- Categories: top-level and subcategories
INSERT INTO categories (name, parent_id) VALUES
  ('Electronics', NULL),
  ('Computers',    1),
  ('Laptops',      2),
  ('Home',        NULL),
  ('Kitchen',      4);

-- Goods linked to each category
INSERT INTO goods (name, category_id, price) VALUES
  ('Smartphone',      1, 699.00),
  ('Gaming Laptop',   3, 1200.00),
  ('Business Laptop', 3,  800.00),
  ('Desktop PC',      2,  600.00),
  ('Refrigerator',    5,  500.00),
  ('Blender',         5,   80.00),
  ('Microwave',       5,  150.00);
```

---

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)
- [Cross Join](./2_cross_join.md)
- [Inner Join](./3_Inner_Join.md)
- [Left Join](./4_Left_Join.md)
- [Right Join](./5_Right_Join.md)
- [Full Oouter Join](./6_Full_Oouter_Join.md)
- [Lateral Join](./7_Lateral_Join.md)
- [Self Join](./8_self_join.md)