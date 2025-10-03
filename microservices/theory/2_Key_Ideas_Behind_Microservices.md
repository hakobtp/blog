# Key Ideas Behind Microservices

When learning about microservices, there are some important ideas to understand. These ideas are often overlooked, but they are essential to know if you want microservices to work well in your projects.

## Independent Deployability

The first key idea is independent deployability. This means you can change one microservice, deploy it, and release it to users without touching any other microservice.

Think of it like updating a single app on your phone without needing to update all your apps at the same time. The concept is simple but tricky to do correctly. If you focus on this idea, many other benefits naturally follow.

To make microservices independently deployable, they must be loosely connected. In other words, one service should work on its own and not rely on the internal details of another service. This requires clear and stable agreements between services, often called APIs. Sharing databases between services is usually a bad idea because it creates tight connections that make independent deployment harder.

For example, imagine you have an online bookstore. One microservice manages books, another manages orders, and a third manages users. If the book service wants to show how many books are available, it should ask the order service for current orders instead of reading the order database directly. This keeps each service independent and easier to update.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Understanding Microservices](./1_Understanding_Microservices.md)