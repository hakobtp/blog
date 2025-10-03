# LazyInitializationException

```info
Author      Ter-Petrosyan Hakob
```
---

Every Java developer who works with Spring Boot and Hibernate eventually meets a frustrating error: the LazyInitializationException. It often appears like this:

```
org.hibernate.LazyInitializationException: could not initialize proxy - no Session
```

At first, it looks scary. But behind it lies a simple truth: you are trying to access lazily loaded data when Hibernate’s session is closed.

<p align="center">
    <img src="./assets/img8.png" alt="img8" width="300"/>
</p>

In this post, we’ll developer building multiple real-world apps struggles with this exception, explores solutions, and learns best practices. 

simple hotel booking application

```java
@Entity
public class Hotel {
    @Id
    private Long id;

    private String name;

    @OneToMany(mappedBy = "hotel", fetch = FetchType.LAZY)
    private List<Room> rooms;
}

@Entity
public class Room {
    @Id
    private Long id;

    private String number;

    @ManyToOne
    private Hotel hotel;
}

```

---

- [Home](./../../README.md)
- [Hibernate Tutorials](./../tutorials.md)
- [Map and Collection of Basic Types](./6_Map_and_Collection_of_Basic_Types.md)

