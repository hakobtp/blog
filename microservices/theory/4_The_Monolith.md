## The Monolith

```info
Author      Ter-Petrosyan Hakob
```

---

We often hear about microservices, but they are usually discussed as the opposite of monolithic architecture. 
To truly understand when microservices might be useful, we first need to understand what a monolith is and why it is sometimes the better choice.

When I say “monolith,” I’m talking about how software is deployed.
If all parts of a system must be deployed together — as one big unit — that system is a monolith.
There are different kinds of monolithic systems, but the most common types are:

- the single-process monolith,
- the modular monolith, and
- the distributed monolith.

## The Single-Process Monolith

The simplest form of monolithic software is when all the code runs in one single process.
You might run multiple copies of that process to handle more users or make the system more reliable, but all of them run the same code base.

For example, imagine a small e-commerce website that handles user accounts, products, and payments — all inside one application.
The same application talks to a single database and shows pages to web or mobile users.

<p align="center">
    <img src="./assets/img3.png" alt="img3" width="300"/>
</p>

This setup is what most developers think of when they hear “monolith.”
However, in real life, many systems are more complex.
A company might have several large applications that are connected tightly — for instance, a sales system and a shipping system that depend on each other.

A `single-process` monolith can still make a lot of sense, especially for small or medium-sized organizations.
It is easier to develop, deploy, and test.
As the company grows, this monolith can also grow — but that can create new challenges, which leads us to the next variation: the modular monolith.

## The Modular Monolith

A modular monolith is still one single application, but it is organized into separate modules inside the same process.
Each module focuses on a different area of the system, such as user management, orders, billing, products, and shipping.
Teams can work on different modules separately, but when it’s time to deploy, all modules are still packaged and deployed together.

<p align="center">
    <img src="./assets/img4.png" alt="img4" width="300"/>
</p>

This idea of dividing a system into smaller parts is not new.
It goes back to structured programming, when programmers learned that splitting code into smaller, focused modules makes it easier to understand and maintain.
Even today, many organizations don’t use this idea properly — they have big, tangled codebases instead of cleanly separated modules.

For many companies, a modular monolith is an excellent middle ground.
If the module boundaries are clear, teams can work in parallel and develop features faster.
At the same time, deployment and monitoring remain much simpler than in a microservices system.

For example, a large online learning platform could have modules for courses, students, and payments.
Each team focuses on one area, but everything still runs together as one application.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Aligning Architecture and Team Organization](./3_Aligning_Architecture_and_Team_Organization.md)