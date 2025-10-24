# Outbox

When building microservices, managing data consistency across services becomes a tricky problem. 
Imagine an `Order Service` that must notify other services—`Inventory`, `Billing`, and `Notifications`—whenever a new order is placed. 
How can we guarantee that messages are reliably sent without introducing complex distributed transactions?


This is where the `Outbox Pattern` comes in. It allows your service to persist internal state (like orders) 
and produce events for other services atomically, using just a single local transaction.

## How the Outbox Pattern Works

The core idea is simple:

1. Your service writes the order data into its database.
2. Alongside, it inserts an event into an `Outbox Events` table.
3. A separate process, often called an outbox relay, reads new events from this table and publishes them to a message broker like Kafka.
4. External services consume these messages and act accordingly.

This guarantees that either both the order and the event are persisted, or neither is—eliminating the risk of missing or duplicate messages.

## Architecture Overview

Here’s a high-level view of how an order request flows through a microservices architecture using the outbox pattern:

<p align="center">
    <img src="./assets/img9.png" alt="img9" width="600"/>
</p>


---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Circuit Breaker](./7_Circuit_Breaker.md)