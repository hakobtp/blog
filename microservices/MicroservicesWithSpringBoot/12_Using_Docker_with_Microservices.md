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
ENTRYPOINT ["java", "org.springframework.boot.loader.launch.JarLauncher"]
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

## Building a Docker Image

Before we can run our microservice in Docker, we need to build the deployment artifact—the fat JAR file—for `course-service`

```bash
pwd
/microservices-with-spring-boot

./gradlew :microservices:course-service:build

ls -l microservices/course-service/build/libs/
-rw-r--r-- 1 htp 1049089 26907426 Nov 26 16:02 course-service-1.0.0-SNAPSHOT.jar

```
Here:

- `./gradlew :microservices:course-service:build` builds the fat JAR.
- `ls -l` shows that the JAR file is created in `build/libs/`.

Next, we create a Docker image and give it the name `course-service`:

```bash
cd microservices/course-service/

ls
Dockerfile  build/  build.gradle  settings.gradle  src/

docker build -t course-service .
```

- Docker uses the `Dockerfile` in the current directory.
- `-t course-service` gives the image a name.
- The image is stored locally in Docker.

To check the image exists:
```bash
docker images | grep course-service

course-service   latest   89df1ba914e4   57 seconds ago   434MB
```

We can now start the microservice:

```bash
docker run --rm -p8080:8080 -e "SPRING_PROFILES_ACTIVE=docker" course-service
```

Here’s what this command does:

- `docker run` – starts the container and shows log output in the Terminal. The Terminal will be locked while the container runs.
- `--rm` – automatically removes the container when we stop it with `Ctrl + C`.
- `-p8080:8080` – maps port `8080` in the container to port `8080` on the host. This makes it 
    possible to access the microservice from outside the container. For example, on macOS with Docker Desktop, 
    it will also forward the port to `localhost`. Only one container can use a host port at a time.
- `-e "SPRING_PROFILES_ACTIVE=docker"` – sets an environment variable to tell Spring Boot which profile to use. Here, we use the docker profile.    
- `course-service` – the name of the Docker image we built.

We can now access the microservice from another Terminal:

```bash
curl localhost:8080/api/v1/courses/1
```

Expected output

```json
{
  "courseId": 1,
  "title": "Introduction to Spring Boot",
  "description": "Learn the basics of Spring Boot 3.",
  "serviceAddress": "******:8080",
  "difficultyLevel": "MEDIUM"
}
```

To see all running containers:

```bash
docker ps
CONTAINER ID   IMAGE            COMMAND                  CREATED         STATUS         PORTS                                         NAMES
bdd38c424630   course-service   "java org.springfram…"   6 minutes ago   Up 6 minutes   0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp   elated_sutherland
```


- `CONTAINER ID` is the unique identifier of the container. This helps to know which container responded to a request.
- `PORTS` shows the mapping between the container and host ports.

To stop the container, press `Ctrl + C` in the Terminal.

## Detached Mode

So far, when we start a Docker container using docker run, the Terminal is locked because the container is running in the foreground. 
This can be inconvenient if we want to keep using the Terminal for other commands.

Docker allows us to run containers in detached mode, meaning the container runs in the background, and the Terminal stays free.

To start a container detached, use the `-d` option. You can also give it a name with `--name` (optional, but helpful):

```bash
docker run -d -p8080:8080 -e "SPRING_PROFILES_ACTIVE=docker" --name CS course-service
```

Use docker ps to see all running containers:

```bash
docker ps

CONTAINER ID   IMAGE            COMMAND                  CREATED          STATUS          PORTS                                         NAMES
9117f2b93d05   course-service   "java org.springfram…"   49 seconds ago   Up 49 seconds   0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp   CS
```

Even in detached mode, you can still see the container logs with:

```bash
docker logs CS -f
```

- `-f` – follows the logs in real-time. The command keeps running, showing new log messages as they appear.
- `--tail 0` – shows only new log messages, skipping old ones.
- `--since 5m` – shows only log messages from the last five minutes.

Try running a request to your microservice while viewing logs:

```bash
curl localhost:8080/api/v1/courses/1
```

When you’re done, stop and remove the container:

```bash
docker rm -f CS
```

- `-f` – forces Docker to stop the container if it’s running and then remove it.

The source code for this article is available [over on GitHub](https://github.com/hakobtp/microservices-with-spring-boot/tree/using_docker_with_microservices){:target="_blank" rel="noopener"}.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Understanding Java Resource Limits in Docker](./11_Understanding_Java_Resource_Limits_in_Docker.md)