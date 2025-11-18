# Creating the API Library

The API project will be packaged as a library, not a standalone application.
This means it won’t have a main class like a typical Spring Boot application. Instead, it only contains 
the code that other microservices can use, such as API interfaces, data models, and exceptions.

Unfortunately, Spring Initializr doesn’t provide an option to create a library project directly.
So we need to create it manually from scratch.

## Structure of the API library

The structure is very similar to a normal Spring Boot application project, with a few differences:

- There is no main application class.
- The `build.gradle` file is slightly different because we are not building a fat JAR.
- We use the Spring Boot platform for dependency management instead of applying the Spring Boot Gradle plugin.

Here is an example `build.gradle` for the API library:

```groovy
plugins {
	id 'java'
	id 'io.spring.dependency-management' version '1.1.7'
}

group = 'com.htp.microservices.api'
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

Here is the project structure:

```
/microservices-with-spring-boot/api/src/main/java/com/htp/microservices/api
├── core
│   ├── chapter
│   │   ├── Chapter.java                # Data model representing a chapter
│   │   └── ChapterService.java         # Interface defining the Chapter REST API
│   │
│   ├── course
│   │   ├── Course.java                 # Data model representing a course
│   │   └── CourseService.java          # Interface defining the Course REST API
│   │
│   └── quiz
│       ├── Quiz.java                   # Data model representing a quiz
│       └── QuizService.java            # Interface defining the Quiz REST API
│
└── exceptions
    ├── BaseException.java              # Common base class for custom exceptions
    ├── InvalidRequestException.java    # Thrown for invalid API requests
    └── NotFoundException.java          # Thrown when a resource is not found

```

### Chapter Record

```java
public record Chapter(
        Long chapterId,
        Long courseId,
        String title,
        String content,
        String serviceAddress
) {
}
```

- Using a record makes the data immutable.
- Records automatically generate constructor, getters, equals, hashCode, and toString.
- Ideal for API models shared across microservices.

### ChapterService Interface

```java

@RequestMapping("/api/v1/chapters")
public interface ChapterService {

    /**
     * Get a chapter by its ID
     * Example: GET /api/v1/chapters/{chapterId}
     */
    @GetMapping("/{chapterId}")
    ResponseEntity<Chapter> getChapterById(@PathVariable Long chapterId);

    /**
     * Get all chapters for a specific course
     * Example: GET /api/v1/chapters/course/{courseId}
     */
    @GetMapping("/course/{courseId}")
    ResponseEntity<List<Chapter>> getChaptersByCourseId(@PathVariable Long courseId);

    /**
     * Create a new chapter
     * Example: POST /api/v1/chapters
     */
    @PostMapping
    ResponseEntity<Chapter> createChapter(@RequestBody Chapter chapter);

    /**
     * Update an existing chapter
     * Example: PUT /api/v1/chapters/{chapterId}
     */
    @PutMapping("/{chapterId}")
    ResponseEntity<Chapter> updateChapter(@PathVariable Long chapterId, @RequestBody Chapter chapter);

    /**
     * Delete a chapter by its ID
     * Example: DELETE /api/v1/chapters/{chapterId}
     */
    @DeleteMapping("/{chapterId}")
    ResponseEntity<Void> deleteChapter(@PathVariable Long chapterId);
}
```

- Base path: `/api/v1/chapters` ensures versioning and clarity.
- CRUD operations fully annotated: `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`.
- Path variables and request bodies annotated properly.
- Ready to be implemented in the Chapter microservice.

### Course Record

```java
public record Course(
        Long courseId,
        String title,
        String description,
        String serviceAddress,
        CourseDifficultyLevel difficultyLevel
) {
}
```

- Immutable, clean, and perfect for an API data model.
- Auto-generates constructor, getters, equals, hashCode, and toString

### CourseService Interface

```java

@RequestMapping("/api/v1/courses")
public interface CourseService {

    /**
     * Get a course by its ID
     * Example: GET /api/v1/courses/{courseId}
     */
    @GetMapping("/{courseId}")
    ResponseEntity<Course> getCourseById(@PathVariable Long courseId);

    /**
     * Get all courses
     * Example: GET /api/v1/courses
     */
    @GetMapping
    ResponseEntity<List<Course>> getAllCourses();

    /**
     * Create a new course
     * Example: POST /api/v1/courses
     */
    @PostMapping
    ResponseEntity<Course> createCourse(@RequestBody Course course);

    /**
     * Update an existing course
     * Example: PUT /api/v1/courses/{courseId}
     */
    @PutMapping("/{courseId}")
    ResponseEntity<Course> updateCourse(@PathVariable Long courseId, @RequestBody Course course);

    /**
     * Delete a course by its ID
     * Example: DELETE /api/v1/courses/{courseId}
     */
    @DeleteMapping("/{courseId}")
    ResponseEntity<Void> deleteCourse(@PathVariable Long courseId);
}
```

- Base path includes API version: `/api/v1/courses`.
- Methods use Spring REST annotations for full CRUD: `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`.
- Fully consistent with ChapterService style.
- Ready to be implemented in the Course microservice.


### Quiz Record

```java
public record Quiz(
        Long quizId,
        Long chapterId,
        String question,
        String correctAnswer,
        String serviceAddress,
        List<String> answerOptions
) {
}
```

- Immutable data model, perfect for API usage.
- Automatically generates constructor, getters, equals, hashCode, and toString.

### CourseService Interface

```java

@RequestMapping("/api/v1/quizzes")
public interface QuizService {

    /**
     * Get a quiz by its ID
     * Example: GET /api/v1/quizzes/{quizId}
     */
    @GetMapping("/{quizId}")
    ResponseEntity<Quiz> getQuizById(@PathVariable Long quizId);

    /**
     * Get all quizzes for a specific course
     * Example: GET /api/v1/quizzes/course/{courseId}
     */
    @GetMapping("/course/{chapterId}")
    ResponseEntity<List<Quiz>> getQuizzesByChapterId(@PathVariable Long chapterId);

    /**
     * Create a new quiz
     * Example: POST /api/v1/quizzes
     */
    @PostMapping
    ResponseEntity<Quiz> createQuiz(@RequestBody Quiz quiz);

    /**
     * Update an existing quiz
     * Example: PUT /api/v1/quizzes/{quizId}
     */
    @PutMapping("/{quizId}")
    ResponseEntity<Quiz> updateQuiz(@PathVariable Long quizId, @RequestBody Quiz quiz);

    /**
     * Delete a quiz by its ID
     * Example: DELETE /api/v1/quizzes/{quizId}
     */
    @DeleteMapping("/{quizId}")
    ResponseEntity<Void> deleteQuiz(@PathVariable Long quizId);
}
```

- Base path includes API version: `/api/v1/quizzes`.
- Fully annotated CRUD operations: `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`.
- Follows the same consistent style as ChapterService and CourseService.
- Ready to be implemented in the Quiz microservice.


## Creating Custom Exceptions

When we build microservices, errors and exceptions are inevitable. Handling them properly is very important because it helps us:

- Give clear messages to clients when something goes wrong.
- Keep our code clean and consistent.
- Avoid repeating the same error-handling logic in multiple places.

In this section, we will create a set of custom exceptions that all microservices can use.

## Why do we need a BaseException?

A `BaseException` acts as the parent for all custom exceptions. By using a base class, we can:

- Group all exceptions in one place for easier management.
- Ensure all exceptions share common features, like a message or a cause.
- Make it easier to handle exceptions globally, for example in a central error handler.

Here’s the `BaseException` class:

```java
public class BaseException extends RuntimeException {

    public BaseException() {
        super();
    }

    public BaseException(String message) {
        super(message);
    }

    public BaseException(String message, Throwable cause) {
        super(message, cause);
    }

    public BaseException(Throwable cause) {
        super(cause);
    }

    public BaseException(String message, Throwable cause, boolean enableSuppression, boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}
```

- It extends `RuntimeException`, so we don’t have to declare it in every method signature.
- All other custom exceptions will inherit from it.

We need two main exceptions for our microservices:

- `InvalidRequestException` – for invalid input or bad requests.
- `NotFoundException` – for cases when a resource is not found.

**InvalidRequestException:**
```java
public class InvalidRequestException extends BaseException {

    public InvalidRequestException(String message) {
        super(message);
    }

    public InvalidRequestException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

**NotFoundException:**
```java
public class NotFoundException extends BaseException {

    public NotFoundException(String message) {
        super(message);
    }

    public NotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

Benefits of this approach

- **Consistency:** All exceptions are grouped and handled in the same way.
- **Clean code:** No repeated error messages or logic scattered across the project.
- **Easy error handling:** We can write a global exception handler to map exceptions to proper HTTP responses (e.g., 400, 404).
- **Reusability:** All microservices can use the same API module exceptions.

## Using the API Module as a Library

Now that we have created a separate API module, we want our microservices to reuse the same data models and service interfaces. 
By including the API module in the Gradle build and adding it as a dependency, we avoid duplicating code and ensure consistency across services.

**Include the API Module in** `settings.gradle`

In your multi-project build, add the API module like this:

```groovy
rootProject.name = 'microservices-with-spring-boot'

include ':api'
include ':microservices:chapter-service'
include ':microservices:course-service'
include ':microservices:course-composite-service'
include ':microservices:quiz-service'
```

- `include ':api'` tells Gradle to include the API project in the multi-project build.
- All other services remain included as before.

**Add the API Module as a Dependency**

Each microservice can now depend on the API module. This allows the service to use the records and interfaces defined in the API project.

Example: `chapter-service/build.gradle`

```groovy
plugins {
	id 'java'
	id 'org.springframework.boot' version '3.5.7'
	id 'io.spring.dependency-management' version '1.1.7'
}

group = 'com.htp.microservices.core.chapter'
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

dependencies {
    implementation project(':api')

	implementation 'org.springframework.boot:spring-boot-starter-actuator'
	implementation 'org.springframework.boot:spring-boot-starter-webflux'
    compileOnly "org.projectlombok:lombok"
    annotationProcessor "org.projectlombok:lombok"
	testImplementation 'org.springframework.boot:spring-boot-starter-test'
	testImplementation 'io.projectreactor:reactor-test'
	testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

tasks.named('test') {
	useJUnitPlatform()
}
```

- `implementation project(':api')` imports the API module into the microservice.
- You can now use `Chapter`, `ChapterService`, and other API classes directly in your service implementation.
- This approach ensures all microservices share the same API definitions, avoiding duplication.
- Works the same for `course-service`, `quiz-service`, and `course-composite-service`—just replace the groupId and service-specific dependencies.


The source code for this article is available [over on GitHub](https://github.com/hakobtp/microservices-with-spring-boot/tree/api_library){:target="_blank" rel="noopener"}.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Api and Util libraries](./4_Api_and_Util_libraries.md)
- [Creating the Util Library](./6_Creating_the_Util_Library.md)