# Service Discovery

When you have many microservices, one big question arises:

**How do clients find a microservice and its available instances?**

## The Problem

Microservices often run in containers, and each container may get a dynamic IP address when it starts. This makes it tricky for clients to know where to send requests.

For example, imagine you have a User Service that exposes an HTTP API. Every time it restarts, it may get a new IP like `10.0.1.15` or `10.0.1.20`. How will the client know the correct address?

Without a solution, requests may fail or go to the wrong instance.