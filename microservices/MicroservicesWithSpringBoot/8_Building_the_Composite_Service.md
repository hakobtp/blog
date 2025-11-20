# Building the Composite Service

Now, it's time to combine everything by adding the `Composite Service`. This service is the one that will call our three core services!

The Composite Service is implemented in two parts:

1. An Integration Component that sends the necessary HTTP requests to the core services.
2. The Composite Service Implementation itself.

We divide the work this way because it simplifies testing. We can test the main service logic separately by replacing the 
Integration Component with a mock (a dummy version that imitates the real one). This makes automated unit and integration testing much easier.

Before we look at the code for these two parts, we need to understand the API classes that the composite microservice will use. 
We also need to see how we use runtime properties to store the network addresses (like the host and port) of the core microservices.

The file structure for the API looks like this:

```
microservices-with-spring-boot/api
└──src/main/java/com/htp/microservices/api/composite
   ├──models
   │  ├──CourseAggregate
   │  ├──ChapterSummary
   │  ├──QuizSummary
   │  └──ServiceAddresses
   └──services
      └──CourseCompositeService
```

These are the data structures (Java records) we will use:


```java
public record ServiceAddresses(
        String course,
        String chapter,
        String quiz,
        String composite) {
}
```

```java
public record QuizSummary(
        Long quizId,
        String question,
        String correctAnswer,
        List<String> answerOptions) {
}
```

```java
public record ChapterSummary(
        Long chapterId,
        String title,
        String content,
        List<QuizSummary> quizzes) {
}
```


The Java interface class `CourseCompositeService` uses the same design pattern as the core services. It looks like this:

```java
public interface CourseCompositeService {

    @GetMapping("/{courseId}")
    CourseAggregate getCourse(@PathVariable Long courseId);
}
```

The model class `CourseAggregate` is a bit more complex than the core data models because it contains fields for lists of chapters and their details:

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

Okay, now let's start the implementation.

## Configuration and Properties

To avoid putting the core service addresses directly into the Java source code, the composite microservice uses a property file. 
This file holds the information needed to find the core services.

The property file, located at `\course-composite-service\src\main\resources\application.yml`, is shown below:

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
  chapter-service:
    host: localhost
    port: 8081
  quiz-service:
    host: localhost
    port: 8003

logging:
  level:
    root: INFO
    se.magnus: DEBUG
```

As we noted, this current setup will be replaced later by a service discovery mechanism. But for now, let's create the configuration class to read these properties.

The configuration class uses Java records and the `@ConfigurationProperties` annotation to easily read the `YAML` data:

```java
@ConfigurationProperties(prefix = "microservices")
public record MicroservicesConfig(
        ServiceConfig courseService,
        ServiceConfig chapterService,
        ServiceConfig quizService
) {

    public record ServiceConfig(String host, int port) {

    }

    public String getCourseServiceUrl() {
        return "http://" + courseService.host() + ":" + courseService.port() + "/api/v1/courses";
    }

    public String getChapterServiceUrl() {
        return "http://" + chapterService.host() + ":" + chapterService.port() + "/api/v1/chapters";
    }

    public String getQuizServiceUrl() {
        return "http://" + quizService.host() + ":" + quizService.port() + "/api/v1/quizzes";
    }
}
```

## The Integration Component

Let's look at the first part of the composite microservice's implementation: the Integration Component, named `CourseCompositeIntegrationService.java`.


It is set up as a Spring Bean using the @Service annotation and implements the API interfaces of the three core services:

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class CourseCompositeIntegrationService implements CourseService, ChapterService, QuizService {
```

The Integration Component uses a utility class from the Spring Framework called `RestTemplate` to handle the actual outgoing HTTP requests to the core microservices.

Before we can use `RestTemplate` in our component, we need to configure it. We do this in the main application class, `CourseCompositeServiceApplication.java`:

```java
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

Now, the Integration Component implements the API methods for the three core services by using the configured `RestTemplate` to make the calls:

We can now inject (or give) the `RestTemplate`, along with a JSON mapper (which helps with error messages) and the 
configuration values from our property file. Here is how this is done

```java
@Slf4j
@Service
@RequiredArgsConstructor
public class CourseCompositeIntegrationService implements CourseService, ChapterService, QuizService {

    private final RestTemplate restTemplate;
    private final MicroservicesConfig microservicesConfig;

    @Override
    public Course getCourseById(Long courseId) {
        var url = microservicesConfig.getCourseServiceUrl() + "/" + courseId;
        return restTemplate.getForObject(url, Course.class);
    }

    @Override
    public List<Chapter> getChaptersByCourseId(Long courseId) {
        var url = microservicesConfig.getChapterServiceUrl() + "?courseId=" + courseId;
        return restTemplate.exchange(url, GET, null,
                new ParameterizedTypeReference<List<Chapter>>() {
                }).getBody();
    }

    @Override
    public List<Quiz> getQuizzesByChapterId(Long chapterId) {
        var url = microservicesConfig.getQuizServiceUrl() + "?chapterId=" + chapterId;
        return restTemplate.exchange(url, GET, null,
                new ParameterizedTypeReference<List<Quiz>>() {
                }).getBody();
    }
}
```

Here are some important points about how the methods are implemented:

1. **For** `getCourseById()`**:** We can use the simple `getForObject()` method in `RestTemplate`. 
    We expect a single Course object. We tell` getForObject()` to map the JSON response directly to the `Course.class` class.
2. **For methods that return lists (like** `getChaptersByCourseId()`**):** We must use the more complex method, `exchange()`. 
    The reason is how Java handles generics (like `List<Chapter>`). At runtime, Java loses the specific type information 
    (it only knows it's a `List`, not a `List<Chapter>`). This is called type erasure.    
    - To solve this, we use the Spring helper class, `ParameterizedTypeReference`. This class is designed to keep the type information (like `List<Chapter>`) at runtime.        
    - Because we need this helper, we must use the more detailed `exchange()` method instead of the simpler `getForObject()` method on `RestTemplate`.

## Composite API Implementation

The final step in building our composite microservice is implementing the API layer, represented by the class `CourseCompositeServiceImpl`.
Before looking at that class, we need a few helper components that prepare and transform data for the composite response. These utilities keep the main service clean and maintainable.

### ServiceAddressResolverUtil

This utility class extracts service-address information from the responses of individual microservices.
Because service addresses may be `null` or missing, this helper ensures that every extraction is safe and returns a valid string.

```java
public interface ServiceAddressResolverUtil {

    static String resolveCourseAddress(Course course) {
        if (course == null) {
            return "";
        }
        var address = course.serviceAddress();
        return address != null ? address : "";
    }

    static String resolveChapterAddress(List<Chapter> chapters) {
        if (CollectionUtils.isEmpty(chapters)) {
            return "";
        }
        return chapters.stream()
                .map(Chapter::serviceAddress)
                .filter(StringUtils::hasLength)
                .findAny()
                .orElse("");
    }

    static String resolveQuizAddress(List<Quiz> quizzes) {
        if (CollectionUtils.isEmpty(quizzes)) {
            return "";
        }
        return quizzes.stream()
                .map(Quiz::serviceAddress)
                .filter(StringUtils::hasLength)
                .findAny()
                .orElse("");
    }
}
```

### CourseMapper

**Purpose:** Converts a `Course` and related service-address information into a `CourseAggregate`.
This mapper keeps the composite service from being overloaded with transformation logic.

```java
@Component
public class CourseMapper {

    public CourseAggregate toCourseAggregate(
            Course course,
            ServiceAddresses serviceAddresses,
            List<ChapterSummary> chapters
    ) {
        return new CourseAggregate(
                course.courseId(),
                course.title(),
                course.description(),
                course.difficultyLevel(),
                serviceAddresses,
                chapters
        );
    }
}
```

### QuizMapper

**Purpose:** Transforms a list of `Quiz` entities into simplified `QuizSummary` DTOs.
It helps the composite service return only the necessary quiz information instead of full domain objects.

```java
@Component
public class QuizMapper {

    public List<QuizSummary> toQuizSummaries(List<Quiz> quizzes) {
        return quizzes.stream()
                .map(this::toQuizSummary)
                .toList();
    }

    public QuizSummary toQuizSummary(Quiz quiz) {
        return new QuizSummary(
                quiz.quizId(),
                quiz.question(),
                quiz.correctAnswer(),
                quiz.answerOptions());
    }

}
```

### ChapterMapper

**Purpose:** Builds a list of `ChapterSummary` objects by combining chapter data with the quizzes that belong to each chapter.
It delegates quiz transformation to `QuizMapper` to keep responsibility separated.

```java
@Component
@RequiredArgsConstructor
public class ChapterMapper {

    private final QuizMapper quizMapper;

    public List<ChapterSummary> toChapterSummaries(
            List<Chapter> chapters,
            Map<Long, List<Quiz>> quizMap
    ) {
        return chapters.stream()
                .map(ch -> new ChapterSummary(
                        ch.chapterId(),
                        ch.title(),
                        ch.content(),
                        quizMapper.toQuizSummaries(quizMap.getOrDefault(ch.chapterId(), List.of()))
                )).toList();
    }
}
```

### ServiceAddressesMapper

**Purpose:** Combines all resolved service addresses—course, chapters, quizzes—and the composite service’s own address into a single `ServiceAddresses` object.
This allows clients to trace where each piece of data originated.

```java
@Component
@RequiredArgsConstructor
public class ServiceAddressesMapper {

    private final ServiceUtil serviceUtil;

    public ServiceAddresses toAddresses(Course course, List<Chapter> chapters, List<Quiz> quizzes) {
        var courseAddress = ServiceAddressResolverUtil.resolveCourseAddress(course);
        var chapterAddress = ServiceAddressResolverUtil.resolveChapterAddress(chapters);
        var quizAddress = ServiceAddressResolverUtil.resolveQuizAddress(quizzes);

        return new ServiceAddresses(
                courseAddress,
                chapterAddress,
                quizAddress,
                serviceUtil.getServiceAddress()
        );
    }
}
```

### CourseCompositeServiceImpl

With all helper classes prepared, we can now implement the main composite API.
This class orchestrates calls to the `course`, `chapter`, and `quiz` microservices, then aggregates the results into a single response.

```java
@Slf4j
@RestController
@RequiredArgsConstructor
@RequestMapping("/api/v1/course-composite")
public class CourseCompositeServiceImpl implements CourseCompositeService {

    private final CourseMapper courseMapper;
    private final ChapterMapper chapterMapper;
    private final ServiceAddressesMapper serviceAddressesMapper;
    private final CourseCompositeIntegrationService integrationService;

    @Override
    public CourseAggregate getCourse(Long courseId) {

        log.info("Fetching composite course structure for courseId={}", courseId);

        var course = integrationService.getCourseById(courseId);
        if (course == null) {
            log.warn("Course not found: {}", courseId);
            throw new NotFoundException("Course not found with id: " + courseId);
        }

        var chapters = integrationService.getChaptersByCourseId(course.courseId());
        log.debug("Fetched {} chapters for course {}", chapters.size(), courseId);


        Map<Long, List<Quiz>> quizMap = chapters.stream()
                .collect(Collectors.toMap(
                        Chapter::chapterId,
                        chapter -> {
                            var chapterId = chapter.chapterId();
                            var quizzes = integrationService.getQuizzesByChapterId(chapterId);
                            log.debug("Found {} quizzes for chapter {}", quizzes.size(), chapterId);
                            return quizzes;
                        }
                ));

        var allQuizzes = quizMap.values().stream().flatMap(List::stream).toList();


        var addresses = serviceAddressesMapper.toAddresses(course, chapters, allQuizzes);
        var chapterSummaries = chapterMapper.toChapterSummaries(chapters, quizMap);
        return courseMapper.toCourseAggregate(course, addresses, chapterSummaries);
    }
}
```

The source code for this article is available [over on GitHub](https://github.com/hakobtp/microservices-with-spring-boot/tree/building_the_composite_service){:target="_blank" rel="noopener"}.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Implementing the Course API](./7_Implementing_the_Course_API.md)    