# Reactive Microservices

Modern software systems need to handle thousands or even millions of requests at the same time. 
The reactive microservices pattern helps developers build systems that stay fast, flexible, and reliable, even under heavy load.

## The Problem

In many traditional Java applications, services communicate synchronously. This means one service sends a request and waits for a reply before doing anything else.

For example, imagine a web app that calls another service to get user details. While waiting for the answer, the thread running that request is blocked — it can’t do anything else. The longer it waits, the fewer requests the system can handle at the same time.

When traffic grows, this blocking behavior leads to slow responses and sometimes even crashes because there are not enough threads available.

## The Solution

To fix this, we can use non-blocking I/O and asynchronous communication.

With non-blocking I/O, the thread doesn’t stop while waiting for a response. It sends the request and continues doing other work. When the response arrives, it handles it using a callback, event, or message queue.

This model is like sending an email instead of making a phone call: you don’t need to wait on the line for the other person to answer; you can do other things while waiting for their reply.

## Key Requirements for Reactive Solutions

To build reactive microservices effectively, developers usually follow these rules:

- Prefer asynchronous communication whenever possible. Send messages or events without waiting for the receiver to finish processing them.
- When synchronous calls are needed, use a reactive framework (for example, Spring WebFlux, Vert.x, or Akka). 
    These frameworks let you handle requests using non-blocking I/O, so the system can serve more users without consuming more threads.
- Design for resilience and self-healing.
    - Resilience means that a service can still send a response even if another service it depends on fails.
    - Self-healing means that once the failing service comes back online, the system automatically reconnects and continues working without manual restart or repair.    

## Reactive Principles

The main ideas behind reactive systems were described in The Reactive Manifesto in 2013.

According to the manifesto, reactive systems are:

- **Message-driven:** they use asynchronous messages for communication.
- **Elastic:** they can scale up or down easily depending on how many users they serve.
- **Resilient:** they can handle failures gracefully and recover automatically.

By combining these features, reactive systems can respond quickly and reliably, even when parts of the system are busy or temporarily broken.

## New Example: A Streaming Service

Imagine a music streaming platform. When a user presses “Play,” the app requests the song from several microservices — one for user data, one for song storage, and one for recommendations.

If these services communicate synchronously, a delay in any one of them could freeze the whole system. But with asynchronous messages, the app can continue working — for example, by showing cached songs or letting the user browse playlists while waiting for the response.

## In Summary

Reactive microservices help us build systems that are:

- Fast — because they don’t block threads.
- Scalable — because they can handle many requests at once.
- Resilient — because they recover automatically from failures.

They’re not just about performance — they’re about creating systems that stay responsive under all conditions.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Edge server](./2_edge_server.md)
- [Central configuration](./4_Central_configuration.md)