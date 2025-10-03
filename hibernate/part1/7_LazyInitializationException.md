# LazyInitializationException

```info
Author      Ter-Petrosyan Hakob
```
---

If you work with Hibernate, you have probably seen the `LazyInitializationException` at least once. It is one of the most common errors when working with databases in Java. Fixing it seems easy, but there is a lot of bad advice online. Some suggested solutions can hide new problems that cause performance issues or wrong data in your application.

In this article, we will cover:

- When Hibernate throws a `LazyInitializationException`
- What not to do to fix it
- Safe ways to fix the exception


## When Does Hibernate Throw a LazyInitializationException?

Hibernate uses a technique called lazy loading. This means it does not load all related data immediately. It waits until you actually need it.

A `LazyInitializationException` happens when Hibernate tries to load related data (like a list of books for an author) outside an active session. This often occurs in the web layer of your application after the transaction has already finished.


<p align="center"> <img src="./assets/img8.png" alt="img8" width="400"/> </p>

For example:

```
```


---

- [Home](./../../README.md)
- [Hibernate Tutorials](./../tutorials.md)
- [Map and Collection of Basic Types](./6_Map_and_Collection_of_Basic_Types.md)

