# Adding Isolated Tests to the Microservices

Before finishing the microservices implementation, it is important to add automated tests. 
These tests help us make sure that our services work as expected.

At this stage, we do not have much business logic, so we do not need many unit tests. Instead, we focus on testing 
the APIs exposed by our microservices. This means we start the services in integration tests with their embedded web 
server and use a test client to send HTTP requests. Then we check if the responses are correct.

Spring `WebFlux` provides a test client called `WebTestClient`, which makes it easy to send requests and check results in a clear, readable way.

## Example: Testing the Composite Course API

Here, we test the course composite API with three main scenarios:

1. Valid course request:
    - Send a `courseId` for an existing course.
    - Expect a `200 OK` HTTP response.
    - Expect a JSON response containing the requested `courseId`, one chapter, and one quiz.
2. Course not found:
    - Send a `courseId` that does not exist.
    - Expect a `404 Not Found` response.
    - Expect a JSON response with relevant error details.
3. Invalid course ID:
    - Send a `courseId` that is invalid.
    - Expect a `422 Unprocessable Entit`y response.
    - Expect a JSON response with an error message explaining the invalid request.    

Here is how the tests are implemented in Java using Spring Boot and Mockito:

```java

@SpringBootTest(webEnvironment = RANDOM_PORT)
class CourseCompositeServiceTest {

    private static final String COURSE_COMPOSITE_URL = "/api/v1/course-composite/";

    private static final Long QUIZ_ID_OK = 10L;
    private static final Long CHAPTER_ID_OK = 11L;

    private static final Long COURSE_ID_OK = 1L;
    private static final Long COURSE_ID_INVALID = 2L;
    private static final Long COURSE_ID_NOT_FOUND = 3L;

    @Autowired
    private WebTestClient client;

    @MockitoBean
    private CourseCompositeIntegrationService integrationService;

    @BeforeEach
    void setUp() {
        var courseOkResponse = new Course(COURSE_ID_OK,
                "Introduction to Spring Boot",
                "Learn the basics of Spring Boot 3.",
                "localhost:9879",
                CourseDifficultyLevel.MEDIUM
        );
        when(integrationService.getCourseById(COURSE_ID_OK)).thenReturn(courseOkResponse);

        var chapter = new Chapter(
                CHAPTER_ID_OK,
                COURSE_ID_OK,
                "Chapter 1: Project Setup with Spring Initializr",
                "This chapter guides you through creating a new Spring Boot project...",
                "localhost:8879");
        when(integrationService.getChaptersByCourseId(COURSE_ID_OK)).thenReturn(singletonList(chapter));

        var quiz = new Quiz(QUIZ_ID_OK, CHAPTER_ID_OK,
                "What is the primary web interface used to bootstrap a Spring Boot project?",
                "Spring Initializr",
                "localhost:7879",
                List.of("Maven Central", "Spring Initializr", "Spring Boot CLI", "Apache Maven"));
        when(integrationService.getQuizzesByChapterId(CHAPTER_ID_OK)).thenReturn(singletonList(quiz));

        when(integrationService.getCourseById(COURSE_ID_NOT_FOUND))
                .thenThrow(new NotFoundException("NOT FOUND: " + COURSE_ID_NOT_FOUND));

        when(integrationService.getCourseById(COURSE_ID_INVALID))
                .thenThrow(new InvalidRequestException("INVALID: " + COURSE_ID_INVALID));
    }

    @Test
    void getCourse() {
        client.get()
                .uri(COURSE_COMPOSITE_URL + COURSE_ID_OK)
                .accept(APPLICATION_JSON)
                .exchange()
                .expectStatus().isOk()
                .expectHeader().contentType(APPLICATION_JSON)
                .expectBody()
                .jsonPath("$.courseId").isEqualTo(COURSE_ID_OK)
                .jsonPath("$.chapters.length()").isEqualTo(1)
                .jsonPath("$.chapters[0].quizzes.length()").isEqualTo(1);
    }

    @Test
    void getCourseNotFound() {
        var uri = COURSE_COMPOSITE_URL + COURSE_ID_NOT_FOUND;
        client.get()
                .uri(uri)
                .accept(APPLICATION_JSON)
                .exchange()
                .expectStatus().isNotFound()
                .expectHeader().contentType(APPLICATION_JSON)
                .expectBody()
                .jsonPath("$.path").isEqualTo(uri)
                .jsonPath("$.timestamp").isNotEmpty()
                .jsonPath("$.httpStatus").isEqualTo(HttpStatus.NOT_FOUND.name())
                .jsonPath("$.message.fieldErrors").isArray()
                .jsonPath("$.message.fieldErrors").isEmpty()
                .jsonPath("$.message.bannerMessage").isEqualTo("NOT FOUND: " + COURSE_ID_NOT_FOUND);
    }

    @Test
    void getCourseInvalidInput() {
        var uri = COURSE_COMPOSITE_URL + COURSE_ID_INVALID;
        client.get()
                .uri(uri)
                .accept(APPLICATION_JSON)
                .exchange()
                .expectStatus().isEqualTo(UNPROCESSABLE_ENTITY)
                .expectHeader().contentType(APPLICATION_JSON)
                .expectBody()
                .jsonPath("$.path").isEqualTo(uri)
                .jsonPath("$.timestamp").isNotEmpty()
                .jsonPath("$.httpStatus").isEqualTo(HttpStatus.UNPROCESSABLE_ENTITY.name())
                .jsonPath("$.message.fieldErrors").isArray()
                .jsonPath("$.message.fieldErrors").isEmpty()
                .jsonPath("$.message.bannerMessage").isEqualTo("INVALID: " + COURSE_ID_INVALID);
    }
}
```

How the Mock Works

- Three constants represent the test course IDs: `COURSE_ID_OK`, `COURSE_ID_NOT_FOUND`, `COURSE_ID_INVALID`.
- The `@MockitoBean` annotation creates a mock for the `CourseCompositeIntegrationService`.
- When `getCourseById()`, `getChaptersByCourseId()`, or `getQuizzesByChapterId()` is called with a valid `courseId`, the mock returns sample data.
- If the course ID is not found, the mock throws a `NotFoundException`.
- If the course ID is invalid, it throws an `InvalidRequestException`.

By using mocks and `WebTestClient`, we can safely test microservice APIs without connecting to real databases or external services. 
This approach makes tests faster and more reliable.

The source code for this article is available [over on GitHub](https://github.com/hakobtp/microservices-with-spring-boot/tree/adding_isolated_tests_to_the_microservices){:target="_blank" rel="noopener"}.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Adding Robust Error Handling to the Composite Client](./9_Adding_Robust_Error_Handling_to_the_Composite_Client.md)