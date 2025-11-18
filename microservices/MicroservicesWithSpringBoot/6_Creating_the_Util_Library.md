# Creating the Util Library

In our microservices project, we want to avoid duplicating code that is used in multiple services. 
That’s why we create a `Util` library, similar to the `API` library.

The `Util` library contains utility classes that help with error handling, service information, and other reusable functionality.
Infrastructure-related information.

## Why do we need a Util Library?

- **Reusability:** Common functionality can be shared across all microservices.
- **Clean code:** Avoid repeating boilerplate code, for example error handling logic.
- **Consistency:** All microservices handle errors and system information in the same way.

## Structure of the Util Library

The project contains these main classes:

1. `GlobalControllerExceptionHandler`
    - Handles exceptions thrown by controllers.
    - Maps Java exceptions like `InvalidRequestException` or `NotFoundException` to proper HTTP status codes and error messages.
2. `ErrorResponse`
    - Represents the structure of the error response sent to the client.
    - Typically contains fields like message, status, and timestamp.
3. `ErrorMessage`
    - Contains the text message for the error.
    - Helps to standardize error messages across all services.
4. `ServiceUtil`
    - Provides utility methods to get service information, such as the hostname, IP address, and port.
    - Useful for debugging or for tracking which instance of a microservice responded to a request.


## Build Configuration

Here is the Gradle configuration for the `Util` library:

```groovy
plugins {
    id 'java'
    id 'io.spring.dependency-management' version '1.1.7'
}

group = 'com.htp.microservices.util'
version = '1.0.0-SNAPSHOT'


java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
    }
}

configurations {
    compileOnly {
        extendsFrom annotationProcessor
    }
}

repositories {
    mavenCentral()
}

ext {
    springBootVersion = '3.5.7'
    lombokVersion = '1.18.42'
}

dependencies {

    implementation project(':api')
    implementation platform("org.springframework.boot:spring-boot-dependencies:${springBootVersion}")

    implementation 'org.springframework.boot:spring-boot-starter-webflux'
    compileOnly "org.projectlombok:lombok:${lombokVersion}"
    annotationProcessor "org.projectlombok:lombok:${lombokVersion}"
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

We need to include the `Util` library in `settings.gradle` so Gradle knows about it:

```groovy
rootProject.name = 'microservices-with-spring-boot'

include ':api'
include ':util'
include ':microservices:chapter-service'
include ':microservices:course-service'
include ':microservices:course-composite-service'
include ':microservices:quiz-service'
```

- Each microservice can add `implementation project(':util')` to its `build.gradle`.
- This allows controllers to use `GlobalControllerExceptionHandler` for error handling.
- Services can use `ServiceUtil` to get hostname, IP, and port information for monitoring or logging.

## Creating Structured Error Messages

When building microservices, it is important to provide clear and structured error responses to clients. 
This helps clients understand what went wrong and which fields caused the problem.

In this section, we will implement `FieldErrorInfo` and `ErrorMessage`, which are reusable classes for managing 
validation and service errors in a consistent way.

The `FieldErrorInfo` class represents an error for a single field, such as a form field or `JSON` property.

```java
import org.springframework.validation.FieldError;

public record FieldErrorInfo(String fieldName, String message) {

    /**
     * Converts a Spring FieldError into our FieldErrorInfo record.
     */
    public static FieldErrorInfo from(FieldError fieldError) {
        return new FieldErrorInfo(
                fieldError.getField(),
                fieldError.getDefaultMessage()
        );
    }
}
```

- This is a Java record, which is an immutable data class.
- The `from()` method is a helper to convert Spring’s `FieldError` into your own error structure.

The `ErrorMessage` class represents an overall error message, which may contain multiple field errors:

```java
@Builder
public record ErrorMessage(
        String bannerMessage,
        @Singular List<FieldErrorInfo> fieldErrors
) {

    public ErrorMessage {
        Objects.requireNonNull(bannerMessage);
        fieldErrors = (fieldErrors == null) ? List.of() : List.copyOf(fieldErrors);
    }

    /**
     * Find a field error by its field name.
     */
    public Optional<FieldErrorInfo> findByFieldName(String fieldName) {
        return fieldErrors.stream()
                .filter(error -> Objects.equals(error.fieldName(), fieldName))
                .findFirst();
    }
}
```

- bannerMessage: a general message describing the error.
- fieldErrors: a list of field-specific errors.
- `@lombok.Builder` Automatically creates a builder pattern for the class.
- `@lombok.Singular`Works with `@Builder` for collections.  Allows you to add items one by one instead of creating the entire list manually.

```java
ErrorMessage error = ErrorMessage.builder()
        .bannerMessage("Validation failed")
        .fieldError(new FieldErrorInfo("title", "Title cannot be empty"))
        .fieldError(new FieldErrorInfo("description", "Description is required"))
        .build();
```

## Creating ErrorResponse for Structured API Errors

Now that we have the `ErrorMessage` class to describe validation or business errors, the next step is to create a 
class that represents the entire error response sent to the client. This class will combine the error message with HTTP status, 
request path, and timestamp, giving a full picture of what went wrong.

```java

@Builder
public record ErrorResponse(
        String path,
        ErrorMessage message,
        HttpStatus httpStatus,
        ZonedDateTime timestamp
) {

    public ErrorResponse {
        Objects.requireNonNull(path, "path cannot be null");
        Objects.requireNonNull(message, "message cannot be null");
        Objects.requireNonNull(httpStatus, "httpStatus cannot be null");
        Objects.requireNonNull(timestamp, "timestamp cannot be null");
    }

    public static ErrorResponse fromException(HttpStatus httpStatus, ServerHttpRequest request, Exception ex) {
        String bannerMessage = (ex != null && ex.getMessage() != null)
                ? ex.getMessage()
                : httpStatus.getReasonPhrase();

        var errorMessage = ErrorMessage.builder()
                .bannerMessage(bannerMessage)
                .build();

        return createBuilder(httpStatus, request)
                .message(errorMessage)
                .build();
    }

    public static ErrorResponse fromMessage(HttpStatus httpStatus, ServerHttpRequest request, ErrorMessage message) {
        return createBuilder(httpStatus, request)
                .message(message)
                .build();
    }

    private static ErrorResponseBuilder createBuilder(HttpStatus httpStatus, ServerHttpRequest request) {
        return ErrorResponse.builder()
                .httpStatus(httpStatus)
                .timestamp(ZonedDateTime.now())
                .path(request.getPath().pathWithinApplication().value());
    }
}
```

- **path:** the `URL` of the request that caused the error.
- **message:** an `ErrorMessage` object containing general and field-specific error details.
- **httpStatus:** the HTTP status code that should be returned (e.g., `400`, `404`).
- **timestamp:** the exact time when the error occurred.

## Creating a Global Exception Handler

When working with multiple microservices, each one may throw different exceptions — for example, when a 
resource is not found, when user input is invalid, or when the server encounters an unexpected error.

If we let every controller handle these cases separately, the code quickly becomes repetitive and hard to maintain.

Instead, we use a global exception handler that intercepts exceptions thrown by any controller and converts them into structured, consistent `ErrorResponse` objects.

Here’s the implementation:

```java

@Slf4j
@RestControllerAdvice
public class GlobalControllerExceptionHandler {

    @ResponseStatus(NOT_FOUND)
    @ExceptionHandler(NotFoundException.class)
    public @ResponseBody ErrorResponse handleNotFoundExceptions(
            ServerHttpRequest request,
            NotFoundException exception
    ) {
        return createHttpErrorResponse(NOT_FOUND, request, exception);
    }

    @ResponseStatus(UNPROCESSABLE_ENTITY)
    @ExceptionHandler(InvalidRequestException.class)
    public @ResponseBody ErrorResponse handleInvalidInputException(
            ServerHttpRequest request,
            InvalidRequestException exception
    ) {
        return createHttpErrorResponse(UNPROCESSABLE_ENTITY, request, exception);
    }

    @ResponseStatus(UNPROCESSABLE_ENTITY)
    @ExceptionHandler(WebExchangeBindException.class)
    public ErrorResponse handleValidationException(ServerHttpRequest request, WebExchangeBindException exception) {
        var fieldErrors = exception.getBindingResult().getFieldErrors().stream()
                .map(FieldErrorInfo::from)
                .toList();

        ErrorMessage errorMessage = ErrorMessage.builder()
                .bannerMessage("Request contains invalid data. Please check field errors.")
                .fieldErrors(fieldErrors)
                .build();

        var errorResponse = ErrorResponse.fromMessage(UNPROCESSABLE_ENTITY, request, errorMessage);

        log.debug("Returning HTTP status: {} for path: {}, message: {}. Field errors: {}",
                errorResponse.httpStatus(), errorResponse.path(),
                errorResponse.message().bannerMessage(), fieldErrors);

        return errorResponse;
    }

    @ResponseStatus(INTERNAL_SERVER_ERROR)
    @ExceptionHandler(Exception.class)
    public ErrorResponse handleGenericException(ServerHttpRequest request, Exception exception) {
        return createHttpErrorResponse(INTERNAL_SERVER_ERROR, request, exception);
    }
    
    private ErrorResponse createHttpErrorResponse(
            HttpStatus status,
            ServerHttpRequest request,
            Exception exception
    ) {
        var er = ErrorResponse.fromException(status, request, exception);
        if (status.is5xxServerError()) {
            log.error("Returning HTTP status: {} for path: {}. Message: {}",
                    er.httpStatus(), er.path(), er.message().bannerMessage(), exception);
        } else {
            log.debug("Returning HTTP status: {} for path: {}. Message: {}",
                    er.httpStatus(), er.path(), er.message().bannerMessage());
        }
        return er;
    }
}
```

**Understanding the Annotations**

- `@RestControllerAdvice` Marks this class as a global exception handler for all REST controllers. 
    Spring will automatically detect it and apply it to every controller in the project.
- `@ExceptionHandler` Defines what type of exception each method handles. For example:    
    - `NotFoundException` → returns `404 Not Found`
    - `InvalidRequestException` → returns `422 Unprocessable Entity`
- `@ResponseStatus` Specifies the HTTP status that will be returned to the client.    
- `@Slf4j` (from Lombok) Automatically creates a logger named `log`.

**Why Do We Need a Global Exception Handler?**

- **Centralization:** One place to manage all exceptions for the entire microservice.
- **Consistency:** Every service returns errors in the same format (ErrorResponse).
- **Less Boilerplate:** Controllers remain clean and focused on business logic.
- **Better Observability:** Each handled exception is logged with useful details.

If a client requests a chapter that doesn’t exist, the API will respond like this:

```json
{
  "path": "/api/v1/chapters/123",
  "message": {
    "bannerMessage": "Chapter not found"
  },
  "httpStatus": "NOT_FOUND",
  "timestamp": "2025-11-07T14:22:33.123Z"
}
```

This makes error responses clear, uniform, and developer-friendly — a best practice for any distributed system.


## Implementing the ServiceUti

In a distributed microservices environment, it’s often useful for a service to know where it’s running — for example, its hostname, IP address, and port number.
This information can be valuable for logging, debugging, and monitoring purposes.

To achieve this, we create a utility class called `ServiceUtil` in our util library.

```java

@Slf4j
@Component
public class ServiceUtil {

    private final String port;
    
    @Getter
    private final String serviceAddress; 

    public ServiceUtil(@Value("${server.port}") String port) {
        this.port = port;
        this.serviceAddress = buildServiceAddress();
    }

    private String buildServiceAddress() {
        try {
            InetAddress localHost = InetAddress.getLocalHost();
            String hostName = localHost.getHostName();
            String hostAddress = localHost.getHostAddress();

            return hostName + "/" + hostAddress + ":" + port;

        } catch (UnknownHostException e) {
            log.warn("Could not determine host address, using defaults.", e);
            return "unknown-host/unknown-ip:" + port;
        }
    }
}
```

The `ServiceUtil` class is a small but essential helper for all microservices in your system.
It provides consistent, easy-to-access infrastructure information that improves observability and debugging.
By calculating the service address once at startup and exposing it through a simple method, we ensure both efficiency and clarity.


The source code for this article is available [over on GitHub](https://github.com/hakobtp/microservices-with-spring-boot/tree/util_library){:target="_blank" rel="noopener"}.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Creating the API Library](./5_Creating_the_API_Library.md)
- [Implementing the Course API](./7_Implementing_the_Course_API.md)