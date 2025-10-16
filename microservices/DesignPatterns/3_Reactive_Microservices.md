## Reactive Microservices

Modern software systems need to handle thousands or even millions of requests at the same time. 
The reactive microservices pattern helps developers build systems that stay fast, flexible, and reliable, even under heavy load.

## The Problem

In many traditional Java applications, services communicate synchronously. This means one service sends a request and waits for a reply before doing anything else.

For example, imagine a web app that calls another service to get user details. While waiting for the answer, the thread running that request is blocked — it can’t do anything else. The longer it waits, the fewer requests the system can handle at the same time.

When traffic grows, this blocking behavior leads to slow responses and sometimes even crashes because there are not enough threads available.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Edge server](./2_edge_server.md)