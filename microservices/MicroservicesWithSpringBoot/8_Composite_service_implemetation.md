## Composite service implemetation

Now, it’s time to tie things together by adding the composite service that will call the three core services!

The implementation of the composite services is divided into two parts: an integration component 
that handles the outgoing HTTP requests to the core services and the composite service implementation itself. The main reason for this division of responsibility is that it simplifies automated unit and 
integration testing; we can test the service implementation in isolation by replacing the integration 
component with a mock.

Before we look into the source code of the two components, we need to take a look at the API classes 
that the composite microservices will use and also learn about how runtime properties are used to 
hold address information for the core microservices.

## API 
In this section, we will take a look at the classes that describe the API of the composite component. 

```
microservices-with-spring-boot/api
|__/src/main/java/com/htp/microservices/api/composite
   |__course
      |--CourseAggregate
      |--CourseCompositeService
      |--ChapterSummary
      |--QuizSummary
      |--ServiceAddresses            
```

```java
public record ServiceAddresses(
        String course,
        String chapter,
        String quiz,
        String composite) {}
```

```java
public record QuizSummary(
        Long quizId,
        String question,
        String correctAnswer, 
        List<String> answerOptions) {}
```

```java
public record ChapterSummary(
        Long chapterId,
        String title,
        String content,
        List<QuizSummary> quizzes) {
}
```

The Java interface class `CourseCompositeService` follows the same pattern that’s used by the  core services and looks as follows:

```java
@RequestMapping("/api/v1/course-composite")
public interface CourseCompositeService {

    @GetMapping("/{courseId}")
    ResponseEntity<CourseAggregate> getCourse(@PathVariable Long courseId);
}
```


The model class `CourseAggregate` is a bit more complex than the core models since it contains 
fields for lists of chapter:

```java
public record CourseAggregate(
        Long courseId,
        String title,
        String description,
        CourseDifficultyLevel difficultyLevel,
        ServiceAddresses serviceAddresses,
        List<ChapterSummary> chapters
) {
}

```

ok now lets start implementation.

To avoid hardcoding the address information for the core services into the source code of the composite microservice, the latter uses a property file where information on how to find the core services 
is stored. The property file, `application.yml`, looks as follows:

```yml
spring:
  application:
    name: course-composite-service

server:
  port: 7000
  error:
    include-message: always

microservices:
  course-service:
    host: localhost
    port: 8082
    protocol: http
  chapter-service:
    host: localhost
    port: 8081
    protocol: http
  quiz-service:
    host: localhost
    port: 8003
    protocol: http

logging:
  level:
    root: INFO
    se.magnus: DEBUG

```

This configuration will, as already noted, be replaced by a service discovery mechanism later on in 
this book until that let's crate config class.

```java
@ConfigurationProperties(prefix = "microservices")
public record MicroservicesConfig(
        ServiceConfig courseService,
        ServiceConfig chapterService,
        ServiceConfig quizService
) {

    public record ServiceConfig(String protocol, String host, int port) {

    }

    public String getCourseServiceUrl() {
        return courseService.protocol() + "://" + courseService.host() + ":" + courseService.port() + "/api/v1/courses";
    }

    public String getChapterServiceUrl() {
        return chapterService.protocol() + "://" + chapterService.host() + ":" + chapterService.port() + "/api/v1/chapters";
    }

    public String getQuizServiceUrl() {
        return quizService.protocol() + "://" + quizService.host() + ":" + quizService.port() + "/api/v1/quizzes";
    }
}
```


t’s look at the first part of the implementation of the composite microservice, the integration component, 
`CourseCompositeIntegration.java`. It is declared as a Spring Bean using the `@Component`
annotation and implements the three core services’ API interfaces:

```java
package com.htp.microservices.core.course_composite.services;
public class CourseCompositeIntegration implements CourseService, ChapterService, QuizService {
....
}
```

The integration component uses a helper class in the Spring Framework, RestTemplate, to perform 
the actual HTTP requests to the core microservices. Before we can inject it into the integration component, we need to configure it. We do that in the main application class, ProductCompositeService
Application.java, as follows:


```java
package com.htp.microservices.core.course_composite;

@SpringBootApplication
@ComponentScan("com.htp")
@EnableConfigurationProperties(MicroservicesConfig.class)
public class CourseCompositeServiceApplication {

    public static void main(String[] args) {
        SpringApplication.run(CourseCompositeServiceApplication.class, args);
    }

    @Bean
    RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

Finally, the integration component implements the API methods for the three core services 
by using RestTemplate to make the actual outgoing calls: full implementation you can find in sourse code 