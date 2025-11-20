# Adding Robust Error Handling to the Composite Client

When the `Composite Service` calls the core microservices, things will not always go smoothly. 

A request may fail for several reasons:
- The target service may not exist or may be temporarily offline
- The client may send invalid data
- The service may respond with an error payload that needs to be interpreted correctly

To handle these problems safely, predictably, and consistently, we add a dedicated error-translation layer 
in the `Composite Service`. This layer converts low-level `HttpClientErrorException` errors coming from `RestTemplate` 
into meaningful, domain-specific exceptions. 

By doing this, we keep the Composite Service clean and avoid leaking technical HTTP errors into higher layers of the system.

Below is how we build this error-handling mechanism step by step.

## Create a Custom Exception

First, we define a new exception class in the shared `util` library.
This exception wraps the structured `ErrorResponse` object returned by the downstream services.

```java
package com.htp.microservices.util.exceptions;

@Getter
@RequiredArgsConstructor
public class CourseCompositeException extends BaseException {

    private final ErrorResponse errorResponse;
}
```

This allows the `Composite Service` to surface errors in a uniform format, regardless of which microservice produced them.

## Extend the Global Exception Handler

Next, we update our `GlobalControllerExceptionHandler` so that it knows how to return the wrapped error to the client:

```java
@ExceptionHandler(CourseCompositeException.class)
public ResponseEntity<ErrorResponse> handleCourseCompositeException(CourseCompositeException exception) {
    var errorResponse = exception.getErrorResponse();
    return ResponseEntity.status(errorResponse.httpStatus())
            .body(errorResponse);
}
```    

This ensures the `Composite API` returns clear, structured `JSON` error messages.

## Introduce the HttpClientErrorHandleService

This class converts low-level `HttpClientErrorException` errors into our custom `CourseCompositeException`.

```java
package com.htp.microservices.core.course_composite.services;

@Slf4j
@Service
@RequiredArgsConstructor
public class HttpClientErrorHandleService {

    private final ObjectMapper objectMapper;

    public void translateHttpClientError(HttpClientErrorException ex) {
        var statusCode = ex.getStatusCode();
        var body = ex.getResponseBodyAsString();
        var resolved = HttpStatus.resolve(statusCode.value());
        if (resolved == null) {
            log.error("Received unknown HTTP status: {}. Body: {}", statusCode, body);
            throw ex;
        }

        if (resolved.is5xxServerError()) {
            log.error("Received server error ({}). Body: {}", statusCode, body);
            throw ex;
        }

        try {
            var errorResponse = objectMapper.readValue(body, ErrorResponse.class);
            log.debug(
                    "Client error ({}). Path: {}. Message: {}",
                    resolved,
                    errorResponse.path(),
                    errorResponse.message() != null ? errorResponse.message().bannerMessage() : "No message"
            );

            throw new CourseCompositeException(errorResponse);

        } catch (JsonProcessingException jsonEx) {
            log.error("Failed to parse error response JSON for status {}. Raw body: {}", statusCode, body);
            throw new RuntimeException("Failed to parse error response from downstream service.", jsonEx);
        }
    }
}
```

This service handles:

- Unknown HTTP statuses
- Server-side errors (5xx)
- JSON parsing of structured error bodies
- Logging for debugging and traceability

## Updating the Integration Component

Finally, we use the new error handler inside the `CourseCompositeIntegrationService`.


```java
package com.htp.microservices.core.course_composite.services;

@Slf4j
@Service
@RequiredArgsConstructor
public class CourseCompositeIntegrationService implements CourseService, ChapterService, QuizService {

    private final RestTemplate restTemplate;
    private final MicroservicesConfig microservicesConfig;
    private final HttpClientErrorHandleService httpClientErrorHandleService;

    @Override
    public Course getCourseById(Long courseId) {
        try {
            var url = microservicesConfig.getCourseServiceUrl() + "/" + courseId;
            return restTemplate.getForObject(url, Course.class);
        } catch (HttpClientErrorException ex) {
            httpClientErrorHandleService.translateHttpClientError(ex);
            return null; // unreachable, but required by compiler
        }
    }

    @Override
    public List<Chapter> getChaptersByCourseId(Long courseId) {
        try {
            var url = microservicesConfig.getChapterServiceUrl() + "?courseId=" + courseId;
            return restTemplate.exchange(url, GET, null,
                    new ParameterizedTypeReference<List<Chapter>>() {
                    }).getBody();
        } catch (Exception ex) {
            log.warn("Got an exception while requesting chapters, return zero chapters: {}", ex.getMessage());
            return List.of();
        }
    }

    @Override
    public List<Quiz> getQuizzesByChapterId(Long chapterId) {
        try {
            var url = microservicesConfig.getQuizServiceUrl() + "?chapterId=" + chapterId;
            return restTemplate.exchange(url, GET, null,
                    new ParameterizedTypeReference<List<Quiz>>() {
                    }).getBody();
        } catch (Exception ex) {
            log.warn("Got an exception while requesting quizzes, return zero quizzes: {}", ex.getMessage());
            return List.of();
        }
    }
}
```

Here’s what’s happening:

- **For course details** → client errors are translated into structured domain exceptions
- **For lists (chapters/quizzes)** → errors degrade gracefully by returning empty lists
- **Unexpected errors** are logged for debugging
- The composite layer remains clean and stable

The source code for this article is available [over on GitHub](https://github.com/hakobtp/microservices-with-spring-boot/tree/composite_client_error_handler){:target="_blank" rel="noopener"}.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Building the Composite Service](./8_Building_the_Composite_Service.md)
- [Adding Isolated Tests to the Microservices](./10_Adding_Isolated_Tests_to_the_Microservices.md)