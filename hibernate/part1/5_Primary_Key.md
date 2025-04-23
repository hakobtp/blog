# Primary Key

```info
Author      Ter-Petrosyan Hakob
```
---

In a relational database, a primary key is one column—or a set of columns—that uniquely identifies each row. 
A primary key must always have a value (no `NULL`s) and no two rows can share the same key. Examples include a customer ID, a phone number, an order reference, or an ISBN.

In JPA, every entity needs an identifier mapped to a primary key. This identifier can be a single field or a combination of fields (a composite key). Once you assign a primary key value to an entity, you should never change it.

A simple primary key in JPA means you use just one field in your entity class. You mark that field with the `@Id` annotation to tell JPA it is the unique identifier. Once set, this value must not change.

Allowed types for a simple primary key include:

- **Java primitive types:** `byte`, `short`, `int`, `long`, `char`
- **Wrapper classes (object versions of those primitives):** `Byte`, `Short`, `Integer`, `Long`, `Character`
- **Arrays of primitives or wrappers:** for example, `int[]`, `Integer[]`
- **Strings and big numbers:** `String`, `BigInteger`
- **Date and time classes:**
    - `java.util.Date`
    - `java.sql.Date`
    - `java.time.LocalDate`
    - `java.time.LocalTime`
    - `java.time.LocalDateTime`
    - `java.time.OffsetTime`
    - `java.time.OffsetDateTime`

Use one of these types for your `@Id` field so that each entity instance has a single, unchangeable key.

When you make an entity, you can set its ID value yourself or let JPA generate it automatically. To use automatic generation, 
add the `@GeneratedValue` annotation and pick one of these four strategies:

- **AUTO:** JPA chooses the best approach for your database. This is the default.
- **IDENTITY:** The database uses an identity column (for example, AUTO_INCREMENT in MySQL) to create each new ID.
- **SEQUENCE:** JPA calls a database sequence object to get the next ID value.
- **TABLE:** JPA keeps the next ID numbers in a special table. Each time you save a new entity, JPA reads and then increments 
    the number in that table. For example, with EclipseLink on H2, you get a table named `SEQUENCE` 
    that has two columns: one for the sequence’s name and one for its current integer value.

```java
@Entity
public class Address {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    ...
}    
```

## Composite Primary Keys

### @EmbeddedId
### @IdClass

---

## 📌 Explore More

- 🏠 [Home](./../../README.md)
- [Secondary Table](./4_SecondaryTable.md)