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

When you map an entity, it’s best to use a single column as its primary key. But sometimes you need a composite key—for example, 
when you work with a legacy database or when business rules require two values (like a date and an order number, or a country code and a timestamp).

To use a composite key:

- Create a key class that holds all the key fields.
- Choose one of two annotations:
    - `@EmbeddedId`
    - `@IdClass`

Both approaches produce the same database tables. The only difference is in how you write your code to load or query the entity.

A class used for a composite primary key must follow these rules:

- **Override equals() and hashCode()**: You must implement these methods so that two key objects are considered 
    equal only when their database key values are exactly the same.
- **Valid field types:** All key fields must use one of the simple types (primitives, wrappers, strings, dates, etc.) listed earlier.
- **Public and no-arg constructor** The class must be public and have a public or protected constructor with no parameters.
- **Serializable (optional)** If you pass key objects outside the persistence layer (for example, to the presentation layer), 
    have the class implement Serializable.

By following these rules, JPA and your code will handle composite keys correctly and safely.

### @EmbeddedId

Let’s create the NewsId composite primary key class:

```java

@Getter
@Setter
@EqualsAndHashCode
@NoArgsConstructor
@AllArgsConstructor

@Embeddable
public class NewsId implements Serializable {

    private String title;
    private String language;
}


```

As you can see, I’m using Lombok annotations to generate the getters, setters, constructors,  `equals()`, and `hashCode()` methods.

In the next example, the `News` entity uses `@EmbeddedId` to include the `NewsId` key class. You do not need the `@Id` annotation here. 
Remember, whatever you use with `@EmbeddedId` must be a class marked with `@Embeddable`.

```java
@Getter
@Setter
@Accessors(chain = true)

@Entity
public class News {

    @EmbeddedId
    private NewsId id;
    private String content;
}
```

JPA generated the following table:

```sql
CREATE TABLE news (
  content  VARCHAR(255),
  language VARCHAR(255) NOT NULL,
  title    VARCHAR(255) NOT NULL,
  PRIMARY KEY (language, title)
);
```

Here, `language` and `title` together form the primary key.

Now let’s write a simple unit test to check that our `News` entity with the composite key works correctly:

```java

class NewsTest {
    private static EntityManager em;
    private static EntityTransaction tx;
    private static EntityManagerFactory emf;

    @BeforeAll
    static void init() {
        emf = Persistence.createEntityManagerFactory("testDB");
        em = emf.createEntityManager();
        tx = em.getTransaction();
    }

    @AfterAll
    static void close() {
        if (em != null) {
            em.close();
        }

        if (emf != null) {
            emf.close();
        }
    }

    @Test
    void test() {
        NewsId newsId = new NewsId("Humboldt penguin marks birthday", "EN");
        tx.begin();
        em.persist(new News().setContent("content").setId(newsId));
        tx.commit();

        tx.begin();
        News news = em.find(News.class, newsId);
        tx.commit();

        assertEquals("content", news.getContent());
        assertEquals(newsId, news.getId());
    }
}
```


### @IdClass

Another way to define a composite key is with the `@IdClass` annotation. With this method, you:

- Create a separate key class with your key fields.
- Repeat those same fields in your entity class and mark each one with `@Id`.

In this example, the `NewsId` class is just a plain Java object (POJO) and needs no JPA annotations.

```java
@Getter
@Setter
@EqualsAndHashCode
@NoArgsConstructor
@AllArgsConstructor
public class NewsId implements Serializable {

    private String title;
    private String language;
}
```

Next, the `News` entity defines its composite key with `@IdClass(NewsId.class)`. Each key field in 
the class (for example, `title` and `language`) is marked with `@Id`. 

```java
@Getter
@Setter
@Accessors(chain = true)

@Entity
@IdClass(NewsId.class)
public class News {

    @Id
    private String title;
    @Id
    private String language;
    private String content;
}
```

Both `@EmbeddedId` and `@IdClass` produce the same table layout:

```sql
CREATE TABLE news (
  content  VARCHAR(255),
  language VARCHAR(255) NOT NULL,
  title    VARCHAR(255) NOT NULL,
  PRIMARY KEY (language, title)
);
```

Next, we write a simple unit test to make sure our `News` entity with its composite key works:

> **NOTE:** Before calling `em.persist(news)`, set both the title and language on your `News` object.

```java
class NewsTest {

    ....

    @Test
    void test() {
        var language = "EN";
        var title = "Humboldt penguin marks birthday";

        var news = new News()
                .setTitle(title)
                .setContent("content")
                .setLanguage(language);

        tx.begin();
        em.persist(news);
        tx.commit();

        NewsId newsId = new NewsId(title, language);

        tx.begin();
        News foundNews = em.find(News.class, newsId);
        tx.commit();

        assertEquals("content", foundNews.getContent());
        assertEquals(newsId.getTitle(), foundNews.getTitle());
        assertEquals(newsId.getLanguage(), foundNews.getLanguage());
    }
}
```

Using `@IdClass` can lead to mistakes because you must repeat each key field in both the key class and the entity, 
and the names and types must match exactly. 


One clear difference appears in JPQL queries:

- With `@IdClass`, you refer to the key fields directly:
    ```sql
    SELECT n.title FROM News n
    ```

- With `@EmbeddedId`, you go through the embedded key object:    
    ```sql
    SELECT n.newsId.title FROM News n
    ```

---

- [Home](./../../README.md)
- [Hibernate Tutorials](./../tutorials.md)
- [Secondary Table](./4_SecondaryTable.md)
- [Map and Collection of Basic Types](./6_Map_and_Collection_of_Basic_Types.md)