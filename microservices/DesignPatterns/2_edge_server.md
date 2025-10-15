# Edge server

When building a system with many microservices, sometimes you want to let external clients access only certain services 
(like a mobile app or web front-end), while keeping other services private inside your system. You also want to protect your 
services from malicious users who might try to abuse them. This is where the Edge Server pattern comes in.

## The Problem

- Some microservices need to be exposed to the outside world.
- Other microservices should remain internal only.
- Exposed services need protection against malicious requests (e.g., hackers trying to access private data).

Without a proper solution, internal services could be accessed directly, which is risky, and managing security individually for each service is complex.

## The Solution

Add an Edge Server to your system. This acts as a gateway for all incoming requests:

- External clients always send requests to the Edge Server.
- The Edge Server decides which requests go to which service.
- It hides internal services from external access.
- It can integrate with Service Discovery to find available service instances and load-balance requests automatically.

Essentially, the Edge Server behaves like a reverse proxy with security features.

## Solution Requirements

- **Hide internal services:** Only allow external access to services meant to be public.
- **Protect exposed services:** Use authentication and authorization, such as:
    - OAuth 2.0
    - OpenID Connect (OIDC)
    - JWT tokens
    - API keys

- **Load balancing:** Forward requests to healthy instances automatically.
- **Centralized security:** No need to configure security for each microservice individually; the Edge Server handles it.

<p align="center">
    <img src="./assets/img2.png" alt="img2" width="500"/>
</p>

Explanation of the diagram:

- `Customer App` → always sends requests to the Edge Server.
- `Edge Server` → forwards requests only to public services like `Catalog` and `Order`.
- Internal services like `Inventory` (tracks stock) and `Recommendation` (suggests books) are not accessible directly from the client.
- All requests pass through the Edge Server, which handles authentication, authorization, and routing.

## Key Points

- The Edge Server acts as a security and traffic gateway.
- It simplifies security, routing, and load balancing for your microservices.
- By using an Edge Server, you protect internal microservices and control access centrally.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Service Discovery](./1_Service_Discovery.md)