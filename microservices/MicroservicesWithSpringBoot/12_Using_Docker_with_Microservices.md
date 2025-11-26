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

> **Tip:** Spring profiles help you use different configurations for development, testing, production, or Docker. 
> Values in a profile replace the default settings. Using YAML, you can put multiple profiles in the same file, separated by `---`.

Here is a simple `Dockerfile` for our microservice:

```Dockerfile
FROM openjdk:17
EXPOSE 8080
ADD ./build/libs/*.jar app.jar
ENTRYPOINT ["java","-jar","/app.jar"]
```

Some important points:

- We use the official OpenJDK 17 image.
- Port `8080` is open so other containers can connect.
- The `ADD` command copies the fat JAR built by Gradle into the image.
- The `ENTRYPOINT` command tells Docker how to start the microservice.

Problems with This Simple Approach

- The full JDK includes compilers and extra tools, which makes the image big and less secure.
- The fat JAR takes time to unpack when starting the container.
- Every time we change code, Docker rebuilds a very large layer, slowing development.

A better way is to split the image into layers:

- Files that change rarely go in first layers.
- Files that change often go in last layers.

This way, Docker can reuse layers that haven’t changed, making builds faster.

## Using a Smaller Java Image

The OpenJDK project doesn’t provide a small Java 17 JRE Docker image. But Eclipse Temurin does. Temurin offers both full JDK and smaller JRE images.

### Handling Fat JAR Files Efficiently

`Spring Boot 2.3.0+` allows us to extract fat JARs into folders, which makes Docker images smaller and faster. By default, Spring Boot creates:

- `dependencies` – all JAR dependencies
- `spring-boot-loader` – classes to start Spring Boot
- `snapshot-dependencies` – snapshot dependencies, if any
- `application` – application code and resources

We can create one Docker layer per folder. This improves caching and speeds up image builds.

Multi-Stage Dockerfile Example:

```Dockerfile
FROM eclipse-temurin:17.0.5_8-jre-focal as builder
WORKDIR extracted
ADD ./build/libs/*.jar app.jar
RUN java -Djarmode=layertools -jar app.jar extract

FROM eclipse-temurin:17.0.5_8-jre-focal
WORKDIR application
COPY --from=builder extracted/dependencies/ ./
COPY --from=builder extracted/spring-boot-loader/ ./
COPY --from=builder extracted/snapshot-dependencies/ ./
COPY --from=builder extracted/application/ ./
EXPOSE 8080
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```

Here’s how it works:

1. **Builder stage**
  - Uses Temurin JRE 17 image.
  - Extracts the fat JAR into the extracted folder.
2. **Final stage**
  - Uses the same Temurin image.  
  - Copies the extracted folders from the builder stage.
  - Each folder becomes a Docker layer.
  - Exposes port `8080`.
  - Starts the microservice using `JarLauncher`.

Using multi-stage builds, we keep the final image small while handling all packaging steps.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Understanding Java Resource Limits in Docker](./11_Understanding_Java_Resource_Limits_in_Docker.md)