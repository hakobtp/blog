# Key Ideas Behind Microservices

When learning about microservices, there are some important ideas to understand. These ideas are often overlooked, but they are essential to know if you want microservices to work well in your projects.

## Independent Deployability

The first key idea is independent deployability. This means you can change one microservice, deploy it, and release it to users without touching any other microservice.

Think of it like updating a single app on your phone without needing to update all your apps at the same time. The concept is simple but tricky to do correctly. If you focus on this idea, many other benefits naturally follow.

To make microservices independently deployable, they must be loosely connected. In other words, one service should work on its own and not rely on the internal details of another service. This requires clear and stable agreements between services, often called APIs. Sharing databases between services is usually a bad idea because it creates tight connections that make independent deployment harder.

For example, imagine you have an online bookstore. One microservice manages books, another manages orders, and a third manages users. If the book service wants to show how many books are available, it should ask the order service for current orders instead of reading the order database directly. This keeps each service independent and easier to update.

## Designing Around Business Domains

Microservices are most effective when they are designed around real-world business domains, not just technical layers. This idea comes from domain-driven design, which helps organize software to match how the business works.

For instance, in a food delivery app, one microservice could handle restaurants, another could handle delivery drivers, and another could manage orders. Each service represents a slice of real-world functionality.

This approach makes it easier to add new features without touching multiple services. Changing one service is faster and less risky than coordinating updates across several services.

Contrast this with older layered architectures, like a typical three-tier system (UI, application logic, database). A simple UI change may require updating multiple layers—making even small updates complicated. By focusing on business functionality instead of technical layers, microservices let you implement changes more efficiently.

## Managing Data and State

Microservices should own their own data instead of sharing databases. Each service decides what information it shares and what it keeps private. This makes services easier to update independently.

This idea is similar to encapsulation in object-oriented programming, where a class hides its internal details from other parts of the program. By hiding internal data, services reduce the chance of causing problems in other parts of the system.

For example, in a fitness tracking app, a "workout" service might store user exercise data, while a "goals" service tracks daily goals. The "goals" service should ask the "workout" service for data rather than directly accessing its database.

## How Big Should a Microservice Be?

Many beginners ask about the size of microservices. The word “micro” can be misleading. A microservice is not defined by lines of code—it’s about being small enough to understand and manage easily.

A practical way to think about size: a service should be as big as you and your team can comfortably understand. For a small team, a microservice might cover just one feature, like user registration. For a large experienced team, it could cover a bigger area, like all user-related features.

The most important thing is not the lines of code, but keeping interfaces small and simple. The smaller the interface between services, the easier it is to change one service without breaking others.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Understanding Microservices](./1_Understanding_Microservices.md)