# Implementing the Course API

Now that we have our shared libraries (`api` and `util`) ready, it’s time to implement the APIs inside our microservices.

Although the structure is similar across all microservices (`course`, `chapter`, and `quiz`), we’ll focus on the `Course Service` as an example.
By the end of this section, you’ll understand how to wire up your own microservice using the shared libraries, handle errors consistently, and expose REST endpoints

First, open the `build.gradle` file in your `course-service` module and add the `api` and `util` projects as dependencies.

```groovy
dependencies {
    implementation project(':api')
    implementation project(':util')

    ...
}

```

The `api` module gives access to interfaces and data models (like `Course` and `CourseService`),
while the util module provides reusable components such as exception handling and `ServiceUtil`.

Since `api` and util `belong` to different packages, Spring Boot won’t automatically detect their beans (like `ServiceUtil`).
To fix this, we add a `@ComponentScan` annotation to the main application class, ensuring Spring looks for components in all relevant packages:

```java
@SpringBootApplication
@ComponentScan("com.htp")
public class CourseServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(CourseServiceApplication.class, args);
    }
}
```

Now we can implement the actual API logic. Create a class named `CourseServiceImpl`.

```java

@Slf4j
@RestController
@RequiredArgsConstructor
@RequestMapping("/api/v1/courses")
public class CourseServiceImpl implements CourseService {


    private final ServiceUtil serviceUtil;
    private final ConcurrentHashMap<Long, Course> courses = new ConcurrentHashMap<>();

    @PostConstruct
    public void init() {
        var serviceAddress = serviceUtil.getServiceAddress();

        courses.put(1L, new Course(1L,
                "Introduction to Spring Boot",
                "Learn the basics of Spring Boot 3.",
                serviceAddress,
                CourseDifficultyLevel.MEDIUM));

        courses.put(2L, new Course(2L,
                "Advanced Java Concurrency",
                "Deep dive into concurrent programming in Java.",
                serviceAddress,
                CourseDifficultyLevel.HARD));

        log.info("Initialized {} courses in the in-memory map.", courses.size());
    }


    @Override
    public Course getCourseById(Long courseId) {
        if (courseId == null || courseId < 1) {
            log.error("Invalid courseId received: {}", courseId);
            throw new InvalidRequestException("Invalid courseId (must be > 0): " + courseId);
        }

        Course course = courses.get(courseId);
        if (course == null) {
            log.warn("Course not found for ID: {}", courseId);
            throw new NotFoundException("Course not found for ID: " + courseId);
        }

        log.debug("Found course with ID {}: {}", courseId, course.title());
        return course;
    }
}
```

Since we don’t have a database yet, all responses are stubbed (hardcoded).
This allows us to test the API layer without setting up persistence.

Finally, create a configuration file named application.yml under:

```
/course-service/src/main/resources/
```

with the following content:

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
```

To build and run the course-service, execute the following commands:

```bash
pwd
.../microservices-with-spring-boot

./gradlew clean build
java -jar course-service/build/libs/*.jar &

```

Once the service is running, test it using curl:

```bash
curl -X GET "http://localhost:8082/api/v1/courses/2"
```

```json
{
  "courseId": 2,
  "title": "Advanced Java Concurrency",
  "description": "Deep dive into concurrent programming in Java.",
  "serviceAddress": "********************:8082",
  "difficultyLevel": "HARD"
}

```

## Important Note about Spring Boot JAR Files

Starting with `Spring Boot 2.5.0`, when you run the `./gradlew build` command, two JAR files are created:

1. **Executable JAR** — the main JAR that can be started with `java -jar`
2. **Plain JAR** — a smaller JAR that only includes compiled class files (no dependencies or startup metadata)

We don’t need the plain JAR in our case.
To simplify running our services, we’ll disable its creation so that we can safely start the application using a wildcard like this:

```bash
java -jar course-service/build/libs/*.jar
```

To disable the plain JAR, add the following lines to the `build.gradle` file of each microservice:

```groovy
jar {
    enabled = false
}
```

This ensures Gradle only builds the single executable JAR we actually use.


The source code for this article is available [over on GitHub](https://github.com/hakobtp/microservices-with-spring-boot/tree/api_implementation){:target="_blank" rel="noopener"}.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Creating the Util Library](./6_Creating_the_Util_Library.md)