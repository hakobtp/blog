# Using Docker with Microservices

Now that we know how Java works inside a container, we can start using Docker with our microservices. 
Before running a microservice as a Docker container, we first need to create a Docker image. To do this, 
we write a file called a `Dockerfile`, which tells Docker how to build the image.

Since a microservice running in a container is isolated from others—it has its own **IP address**, **hostname**, 
and **ports**—it needs a different configuration than when it runs on the same computer as other microservices.

For example, because the microservices are no longer on the same host, we don’t have to worry about port conflicts. 
We can use the default port `8080` for all microservices. However, if one microservice needs to communicate with another, 
we cannot use `localhost` anymore, because each container has its own network address.

To manage these differences, we use Spring profiles, which let us define configuration for different environments. 
We will create a new profile called `docker` for running microservices inside Docker containers.

Add this at the end of the `application.yml` file:

```yml
server:
  port: 8082
spring:
  application:
    name: course-service
logging:
  level:
    root: INFO
    com.htp: DEBUG
---
spring.config.activate.on-profile: docker
server.port: 8080
```

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Understanding Java Resource Limits in Docker](./11_Understanding_Java_Resource_Limits_in_Docker.md)