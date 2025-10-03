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

<p align="center"> <img src="./assets/img8.png" alt="img8" width="300"/> </p>

In this post, we’ll explore what this exception means, why it happens, and how to solve it with practical examples from different kinds of applications.


---

- [Home](./../../README.md)
- [Hibernate Tutorials](./../tutorials.md)
- [Map and Collection of Basic Types](./6_Map_and_Collection_of_Basic_Types.md)

