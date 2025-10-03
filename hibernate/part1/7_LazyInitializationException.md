# LazyInitializationException

```info
Author      Ter-Petrosyan Hakob
```
---

If you work with Spring Boot and Hibernate, you may have seen this error:

```yaml
org.hibernate.LazyInitializationException: could not initialize proxy - no Session
```

At first, it can look confusing or scary. But the reason is simple: you are trying to access lazy-loaded data after Hibernate has closed the session.

<p align="center"> <img src="./assets/img8.png" alt="img8" width="400"/> </p>

In this post, we’ll explore what this exception means, why it happens, and how to solve it with practical examples from different kinds of applications.

## Lazy Loading Example: Library System

Imagine we have a Library with Books. Each library can have many books. We want to load libraries lazily — only fetching books when we need them.

```java
@Entity
public class Library {
    @Id
    private Long id;

    private String name;

    @OneToMany(mappedBy = "library", fetch = FetchType.LAZY)
    private List<Book> books;
}

@Entity
public class Book {
    @Id
    private Long id;

    private String title;

    @ManyToOne
    private Library library;
}
```

Here, books are lazy-loaded. Hibernate will not fetch them immediately when we load a library.

The Problem

```java
Library library = libraryRepository.findById(1L).get();
System.out.println("Library name: " + library.getName());
System.out.println("Books: " + library.getBooks().size());
```

If the session closes after `findById`, calling `getBooks()` will throw:

```yaml
LazyInitializationException: could not initialize proxy - no Session
```

Why? Hibernate only runs this query when the lazy collection is accessed:

```sql
select b.id, b.title from Book b where b.library_id = 1;
```

If the session is closed, Hibernate cannot fetch the books, so the exception occurs.


## Solution 1: Use DTOs (Data Transfer Objects)

We can create a LibraryDTO that fetches data in a single query.

```java
public class LibraryDTO {
    private String name;
    private List<String> bookTitles;

    public LibraryDTO(String name, List<String> bookTitles) {
        this.name = name;
        this.bookTitles = bookTitles;
    }
}
```

Repository query:

```java
@Query("SELECT new com.example.LibraryDTO(l.name, b.title) " +
       "FROM Library l JOIN l.books b WHERE l.id = :id")
LibraryDTO findLibraryWithBooks(@Param("id") Long id);

```

SQL generated:

```sql
select l.name, b.title
from Library l
join Book b on l.id = b.library_id
where l.id = 1;
```

- **Pros:** No exception, works outside session.
- **Cons:** Extra DTO class, mapping work.

---

- [Home](./../../README.md)
- [Hibernate Tutorials](./../tutorials.md)
- [Map and Collection of Basic Types](./6_Map_and_Collection_of_Basic_Types.md)

