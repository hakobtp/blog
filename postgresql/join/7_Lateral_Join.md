# 🔗 Lateral Join

```info
Author      Ter-Petrosyan Hakob
```

---

A **LATERAL JOIN** is a type of join in SQL that allows you to join a table with a subquery, where the subquery is run for each row of the main table. 
The subquery is executed before joining the rows and the result is used to join the rows. With this join mode, you can use information from one table to filter or process data from another table.

To find all categories that have at least one good priced over 699:

```sql
SELECT c.*
FROM categories AS c
WHERE EXISTS (SELECT 1
              FROM goods AS g
              WHERE g.category_id = c.id AND g.price > 699);

 id |  name   | parent_id 
----+---------+-----------
  3 | Laptops |         2
  5 | Kitchen |         4
(2 rows)

```

If you also want to return the `price` and `name` of each matching `good`, use a **LATERAL JOIN**:

```sql
SELECT c.id,
       c.name    AS category_name,
       good.price,
       good.name AS good_name
FROM categories AS c JOIN LATERAL (
    SELECT g.price, g.name FROM goods AS g WHERE g.category_id = c.id AND g.price > 699
) AS good ON TRUE;


 id | category_name |  price  |    good_name    
----+---------------+---------+-----------------
  3 | Laptops       | 1200.00 | Gaming Laptop
  3 | Laptops       |  800.00 | Business Laptop
  5 | Kitchen       |  700.00 | Microwave
(3 rows)

```

In SQL, every **JOIN** needs an **ON** clause (or a **USING** clause) that tells the engine how to match rows between the two sources. 
With a **LATERAL JOIN**, you’re already filtering inside the subquery (e.g. `WHERE g.category_id = c.id AND g.price > 699`), so you 
don’t need any additional join condition. You still must write something after **ON**, so you use a condition that’s always **true**:

## Why LATERAL?

Unlike **EXISTS**, which only tests for presence, **LATERAL JOIN** lets you pull columns from the subquery directly into your main result set.

---

## 📌 Explore More

- 🏠 [Home](./../../README.md)
- 📚 [PostgreSql Tutorials](./../tutorials.md)