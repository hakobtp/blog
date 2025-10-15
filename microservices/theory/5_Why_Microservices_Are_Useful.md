# Why Microservices Are Useful

```info
Author      Ter-Petrosyan Hakob
```

---

Microservices bring many benefits, especially when compared to other distributed systems. 
They are designed with clear service boundaries, using ideas like information hiding and domain-driven design, 
which help organize the system better. By doing this, microservices often make distributed systems more effective and easier to manage.

## Using Different Technologies

Microservices allow us to use different technologies for different parts of a system. 
Instead of using a single technology for everything, we can choose the best tool for each task.

For example, imagine an online bookstore: we could store customer orders in a relational database, 
while keeping book reviews in a document database for faster searches. This gives us a flexible and heterogeneous system.

We can also try new technologies safely. In a monolithic system, changing a programming language or database can affect the entire system. In a microservices system, we can experiment with one service first, limiting the risk of failure.

However, using many technologies adds some complexity. Some companies set rules to simplify things. For instance, a company may mostly use Java because they understand it well and have tools that work with it, but they still mix in other technologies where needed. Hiding the internal technology of services also makes upgrades safer — we can update one microservice without affecting others.

## Making Your System Stronger

Microservices can make systems more robust, meaning the system can handle failures better.

A useful idea here is the bulkhead: if one part fails, it doesn’t crash the whole system. In a monolithic system, a single failure can stop everything. In microservices, we can isolate problems and keep the rest of the system running.

For example, in a food delivery app, if the payment service fails, the menu and search features can still work while engineers fix the payment issue.

But microservices come with new risks. Networks and machines can fail, so we must design the system to handle these failures without affecting users.

## Growing Your System Easily

In large monolithic applications, scaling means scaling everything together. But with microservices, we can scale only the parts that need it. 
For example, imagine a video streaming platform. If only the video recommendation service is slow, we can scale just that service, saving resources on other parts of the system.

Using cloud services like AWS, we can scale services on demand, which saves money and improves performance.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [The Monolith](./4_The_Monolith.md)