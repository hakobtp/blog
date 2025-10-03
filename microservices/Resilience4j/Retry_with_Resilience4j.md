# Retry with Resilience4j

```info
Author      Ter-Petrosyan Hakob
```

---

In this article, we first give a short introduction to [Resilience4j](https://resilience4j.readme.io/){:target="_blank" rel="noopener"}. 
Then we look closely at its Retry module. You will learn when to use retries, how to set them up, and which features 
[Resilience4j](https://resilience4j.readme.io/){:target="_blank" rel="noopener"}` offers.

## Introduction to Resilience4j

When applications talk to each other over a network, many things can go wrong. A request might time out, 
connections can break, or an upstream service may be down. Sometimes, a sudden load of requests can slow or crash a service.

[Resilience4j](https://resilience4j.readme.io/){:target="_blank" rel="noopener"} is a Java library that helps you make your applications more reliable. 
It gives you tools to handle failures and keep your system running smoothly.

Let’s have a quick look at the modules and their descriptions:

## Resilience4j Modules and Their Descriptions

| Module         | Description                                                 |maven artifactId            |
| -------------- | ----------------------------------------------------------- |:---------------------------|
| Retry          | Automatically try a failed call again                       |resilience4j-retry          |
| RateLimiter    | Limit how often a call can be made in a given time          |resilience4j-ratelimiter    |
| TimeLimiter    | Set a maximum time for a call to finish                     |resilience4j-timelimiter    |
| Circuit Breaker| Stop calling or use a fallback when failures keep happening |resilience4j-circuitbreaker |
| Bulkhead       | Limit how many calls can run at the same time               |resilience4j-bulkhead       |
| Cache          | Save and reuse results of expensive calls                   |resilience4j-cache          |


Let’s set up a Maven project. Please note that I’ve added the `resilience4j-all` dependency, 
but if you prefer, you can include only the modules you need.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.hakobtp.blog</groupId>
    <artifactId>blog</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <properties>
        <java.version>17</java.version>
        <lombok.version>1.18.38</lombok.version>
        <resilience4j.version>2.3.0</resilience4j.version>
        <spring-cloud.version>2024.0.1</spring-cloud.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>io.github.resilience4j</groupId>
            <artifactId>resilience4j-all</artifactId>
            <version>${resilience4j.version}</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>${lombok.version}</version>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>

```

## What Is a Remote Operation?

A **remote operation** is any request your application makes over a network. Common examples include:

- Sending an HTTP request to a REST API  
- Calling a remote procedure (RPC) or web service  
- Reading or writing data in a database or object store  
- Sending or receiving messages from a broker (RabbitMQ, Kafka, etc.)  

### What Happens When It Fails?

If a remote operation fails, you have two choices:

1. **Fail fast** – immediately return an error to the client.  
2. **Retry** – try the operation again.  

Retrying can hide temporary issues so that clients don’t notice a hiccup. 

### Which Option to Choose?

Deciding between failing fast and retrying depends on:

- **Error type**  
  - **Transient errors** are short‑lived. A retry often succeeds.  
    - Example: request throttled, network timeout, temporary service outage  
  - **Permanent errors** cannot be fixed by retrying.  
    - Example: hardware failure, HTTP 404 Not Found  

- **Operation type**  
  - **Idempotent operations** can run multiple times safely (e.g., reading data).  
  - **Non‑idempotent operations** may cause unwanted side effects if repeated (e.g., money transfers).  

- **Client type**  
  - **Applications** (cron jobs, background processes) can wait longer.  
  - **People** usually expect a quick response.  

- **Use case**  
  - Some tasks need high reliability more than speed (e.g., booking flights, transferring money).  

### Why Idempotency Matters

If a service processed your request but failed to send back the answer, a retry might be treated as a brand‑new request. 
For safe retries, make sure the operation is **idempotent**—the system can handle the same request more than once without side effects.

### Balancing Speed and Reliability

- **Faster feedback** is better for human users. Failing quickly lets you show an error message right away.  
- **More reliability** can matter more for critical tasks. In those cases, you might:

  1. **Acknowledge** the request immediately (so the user knows it was received).  
  2. **Perform retries** in the background.  
  3. **Notify** the user when the work is done.  

This way, you keep both reliability and a good user experience.


## Resilience4j Retry Module

Resilience4j’s retry feature uses three simple parts:

- **RetryConfig:** Defines retry rules, such as how many attempts to make and how long to wait between them.
- **RetryRegistry:** Holds one or more `RetryConfig` objects and creates named `Retry` instances from them.
- **Retry:** Uses its `RetryConfig` to wrap your code (a lambda, method reference, or functional interface) so that Resilience4j 
    will automatically retry the operation on failure.

With these building blocks, you can easily add retry logic around any remote call.


## Simple Retry

In a simple retry, the operation is retried if a `RuntimeException` is thrown during the remote call. We can configure the number of attempts, how long to wait between attempts etc.:

```java
private static void demonstrateRetryWithUncheckedException() {
    System.out.println("\n=== Resilience4j Retry Demonstration ===");

    // 1. Define retry rules:
    //    - maxAttempts(3): try up to 3 times
    //    - waitDuration(2 seconds): pause 2 seconds between tries
    //RetryConfig config = RetryConfig.ofDefaults(); // you can use default settings
    RetryConfig config = RetryConfig.custom()
            .maxAttempts(3)
            .waitDuration(Duration.of(2, SECONDS))
            .build();

    // 2. Create a registry to manage Retry instances
    RetryRegistry retryRegistry = RetryRegistry.of(config);

    // 3. Create or retrieve a Retry named "userServiceRetry" using our config
    Retry retry = retryRegistry.retry("userServiceRetry", config);

    // 4. Prepare the remote call as a Supplier (lambda)
    //    Here we search users with firstName = "Bob"
    UserQuery query = UserQuery.builder()
                               .firstName("Bob")
                               .build();
    Supplier<List<User>> remoteCallSupplier = () -> userService.search(query);

    // 5. Decorate the Supplier so it will retry on failure
    Supplier<List<User>> retryingUserSearch = Retry.decorateSupplier(retry, remoteCallSupplier);

    try {
        // 6. Execute the decorated call
        List<User> users = retryingUserSearch.get();
        //    Print each returned user
        users.forEach(System.out::println);
    } catch (Exception e) {
        // 7. Handle the case where all retry attempts failed
        System.err.println("All retries failed: " + e.getMessage());
    }
}

```

We would use `decorateSupplier()` if we wanted to create a decorator and 
re-use it at a different place in the codebase. If we want to create it and immediately execute it, we 
can use `executeSupplier()` instance method instead:

- `Retry.decorateSupplier()` gives you a reusable function you can call multiple times.
- `retry.executeSupplier()` runs the supplier immediately with retry logic under the hood.

```java
var query = UserQuery.builder().firstName("Bob").build();
List<User> users = retry.executeSupplier(()-> userService.search(query));
```

### Checked Exceptions

Now, suppose we need to retry operations that can throw both checked and unchecked exceptions. For example, calling  `userService.searchThrowingException()` 
may throw a checked exception. Because Java’s Supplier interface doesn’t allow checked exceptions, this will cause a compiler error. 
Instead, you should use Resilience4j’s

```java
io.github.resilience4j.core.functions.CheckedSupplier
```

```java
private static void demonstrateRetryWithCheckedException() {
    ...

var query = UserQuery.builder().firstName("Bob").build();
CheckedSupplier<List<User>> remoteCallSupplier = () -> userService.searchThrowingException(query);
CheckedSupplier<List<User>> retryingFlightSearch = Retry.decorateCheckedSupplier(retry, remoteCallSupplier);

...
}
```

If you prefer not to use `Supplier`, the Retry module offers other decorator methods, for example: 
`decorateFunction()`, `decorateCheckedFunction()`, `decorateRunnable()`, `decorateCallable()` and etc.

The plain `decorate*` methods only retry on unchecked exceptions (`RuntimeException`).  
The `decorateChecked*` variants will retry on both checked (`Exception`) and unchecked exceptions.

### Conditional Retry

In real applications, we don’t always want to retry every error. For example, if an `AuthenticationException` happens, retrying won’t help. When calling an HTTP service, you can:

- Check the HTTP status code (for example, 401 Unauthorized)  
- Look for a specific error code in the response body  

Use these checks to decide whether to retry. Next, we’ll see how to set up conditional retries in Resilience4j.

```java
private static void demonstrateRetryPredicateConfig() {
    var errorCodes = List.of(5678, 5679);
    RetryConfig config = RetryConfig.<UserWitErrorCode>custom()
            .maxAttempts(3)
            .waitDuration(Duration.of(2, SECONDS))
            .retryOnResult(userWithErrorCode -> errorCodes.contains(userWithErrorCode.errorCode()))
            .build();

    RetryRegistry retryRegistry = RetryRegistry.of(config);
    Retry retry = retryRegistry.retry("userServiceRetry", config);
    Supplier<UserWitErrorCode> userWitErrorCodeSupplier = retry.decorateSupplier(() -> userService.findById(1));
    
    try {
        var user = userWitErrorCodeSupplier.get().user();
        System.out.println(user);
    } catch (Exception e) {
        System.err.println("All retries failed: " + e.getMessage());
    }

}
```    

In this example, `userService.findById(1)` returns a record:

```java
public record UserWithErrorCode(Integer errorCode, User user) { }
```

We use this line in our retry configuration:

```java
.retryOnResult(userWithErrorCode -> errorCodes.contains(userWithErrorCode.errorCode()))
```

This means:

- If the returned `errorCode` is in the list `[5678, 5679]`, Resilience4j treats it as a temporary error. 
    It waits 2 seconds, then retries the call—up to 3 attempts in total.
- If the `errorCode` is not in that list, it stops retrying immediately.

In other words, the `retryOnResult` predicate lets us retry only for specific error codes that we know are temporary.

### Conditional Retry Based on Exception Types

Sometimes we want to retry on general failures but skip retrying for certain cases. For example:

- We throw `UserServiceException` for any unexpected error in the user service.  
- But if a `UserNotFoundException` happens, retrying won’t help because the user does not exist.

We can configure this with `retryExceptions` and `ignoreExceptions`:

```java
RetryConfig config = RetryConfig.<UserWithErrorCode>custom()
    .maxAttempts(3)
    .waitDuration(Duration.ofSeconds(2))
    .retryExceptions(UserServiceException.class)   // retry on general service errors
    .ignoreExceptions(UserNotFoundException.class) // do not retry if user not found
    .build();
```

- `retryExceptions(...)` lists the exception types that trigger a retry (including subclasses).
- `ignoreExceptions(...)` lists the exception types that should not be retried.
- Any other exception (for example, `IOException`) will not be retried, because it is neither in `retryExceptions` nor in `ignoreExceptions`.


### Conditional Retry with a Predicate

Sometimes even a single exception type needs extra checks. For example, a `LimitException` may include an error code. 
We only want to retry when that code is `34565`:

```java
Predicate<Throwable> limitPredicate = ex -> {
    if (ex instanceof LimitException le) {
        return le.getCode() == 34565; // retry only for code 34565
    }
    return false;
};

RetryConfig conditionalConfig = RetryConfig.<UserWithErrorCode>custom()
            .maxAttempts(3)
            .waitDuration(Duration.ofSeconds(2))
            .retryOnException(limitPredicate)
            .build();

```

- `retryOnException(...)` takes a test (`Predicate<Throwable>`) that returns true when we should retry.
- You can make this test as simple or as detailed as you need (checking error codes, message text, etc.).

With these options, you decide exactly which errors and conditions cause a retry.


### Backoff Strategies

Until now, we used a fixed wait time between retries. In most cases, it is better to **increase** the delay after each attempt. 
This is called **backoff**, and it gives the remote service more time to recover if it is overloaded.

Resilience4j uses an **IntervalFunction** to control backoff. An `IntervalFunction` takes the attempt number (1, 2, 3, …) 
and returns the wait time in milliseconds.

Use `ofRandomized(baseDelay)` to add randomness around a base delay. By default, it uses a **randomization factor** of `0.5`:

```java
RetryConfig config = RetryConfig.custom()
    .maxAttempts(4)
    .intervalFunction(IntervalFunction.ofRandomized(2000))
    .build();
```
Base 
- delay: 2000 ms
- Factor: 0.5 (default)
- Actual delay: between
    - 2000 − 2000×0.5 = 1000 ms
    - 2000 + 2000×0.5 = 3000 ms

You can change the factor with `ofRandomized(baseDelay, randomizationFactor)`.

Use `ofExponentialBackoff(initialDelay, multiplier)` so each wait time doubles (or more) each try:

```java
RetryConfig config = RetryConfig.custom()
    .maxAttempts(6)
    .intervalFunction(IntervalFunction.ofExponentialBackoff(1000, 2))
    .build();
```

- Attempt 1: 1000 ms
- Attempt 2: 1000 × 2 = 2000 ms
- Attempt 3: 2000 × 2 = 4000 ms
- …and so on.

Combine both strategies with `ofExponentialRandomBackoff(initialDelay, multiplier)`. This adds randomness to the exponential delays.

You can write your own function, for example, to add a small random jitter:

```java
IntervalFunction jitterBackoff = attempt ->
    500L * (long) Math.pow(2, attempt - 1)
    + ThreadLocalRandom.current().nextLong(100);

RetryConfig config = RetryConfig.custom()
    .maxAttempts(6)
    .intervalFunction(jitterBackoff)
    .build();
```

- Base exponential delay: 500 ms, doubled each attempt
- Plus up to 100 ms random extra delay

With IntervalFunction, you have full control over how delays grow. Pick the strategy that best fits your scenario.


## Asynchronous Retries

So far, our examples have used synchronous calls. Now let’s see how to retry **asynchronous** operations.

Imagine we search for users on another thread:

```java
CompletableFuture
    .supplyAsync(() -> userService.search(query))
    .thenAccept(System.out::println);
```

Here, `supplyAsync()` runs `search(query)` in a separate thread. When it finishes, `thenAccept()` prints the list of users.

To add retries, use the `executeCompletionStage()` method on your `Retry` instance. It needs:

- A `ScheduledExecutorService` for scheduling retry attempts.
- A `Supplier<CompletionStage<…>>` that wraps the async call.

For example:

```java
// 1. Create a scheduler for retries
ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();

// 2. Wrap the async user search in a Supplier
Supplier<CompletionStage<List<User>>> completionStageSupplier =
    () -> CompletableFuture.supplyAsync(() -> userService.search(query));

// 3. Execute with retry logic
retry.executeCompletionStage(scheduler, completionStageSupplier)
    .thenAccept(System.out::println);
```              

- **Step 1:** We use a single-threaded scheduler here, but in real apps, you would use a shared pool, e.g.: `Executors.newScheduledThreadPool(n)`.
- **Step 2:** The supplier returns a `CompletionStage` that will run the search.
- **Step 3:** `executeCompletionStage()` decorates and runs the async call, retrying if needed. 
    The returned `CompletionStage` still lets you use `thenAccept()` or other callbacks.

This approach makes your asynchronous code resilient, automatically retrying failed operations without blocking the main thread.


### Events

By default, retrying is a `“black box”`—we don’t see when an attempt fails or when a retry happens. 
To log these details, Resilience4j lets you listen to **Retry events**. You can use the `EventPublisher` to run code on each retry, success, or error.

```java
// Get the publisher from your Retry instance
Retry.EventPublisher publisher = retry.getEventPublisher();

// Log each retry attempt
publisher.onRetry(event ->
    System.out.println("Retry #" 
        + event.getNumberOfRetryAttempts() 
        + ", wait " 
        + event.getWaitInterval().toMillis() 
        + " ms")
);

// Log when it finally succeeds
publisher.onSuccess(event ->
    System.out.println("Succeeded after " 
        + event.getNumberOfRetryAttempts() 
        + " attempts")
);

// Log if all retries fail
publisher.onError(event ->
    System.err.println("Failed after " 
        + event.getNumberOfRetryAttempts() 
        + ": " 
        + event.getLastThrowable().getMessage())
);
```

You can also watch the `RetryRegistry` to see when `Retry` instances are added or removed:

```java
RetryRegistry.EventPublisher<Retry> registryPub = retryRegistry.getEventPublisher();


registryPub.onEntryAdded(entry ->
    System.out.println("Added retry: " + entry.getAddedEntry().getName())
);

registryPub.onEntryRemoved(entry ->
    System.out.println("Removed retry: " + entry.getRemovedEntry().getName())
);

```


### Metrics

Resilience4j tracks how many calls:

- Succeed on the first try
- Succeed after one or more retries
- Fail without any retry
- Fail even after retries

To collect these metrics, use Micrometer. First, bind your `RetryRegistry` to a `MeterRegistry`:


```java
MeterRegistry meterRegistry = new SimpleMeterRegistry();
TaggedRetryMetrics.ofRetryRegistry(retryRegistry).bindTo(meterRegistry);
```

Then you can read and print each metric:

```java
Consumer<Meter> meterConsumer = meter -> {
        String desc = meter.getId().getDescription();
        String metricName = meter.getId().getTag("kind");
        Double metricValue = StreamSupport.stream(meter.measure().spliterator(), false)
                .filter(m -> m.getStatistic().name().equals("COUNT"))
                .findFirst()
                .map(Measurement::getValue)
                .orElse(0.0);
        System.out.println(desc + " - " + metricName + ": " + metricValue);
};
meterRegistry.forEachMeter(meterConsumer);
```
In production, send these metrics to a monitoring system (Prometheus.) for dashboards and alerts.

---

```java
private static void demonstrateRetryWithCheckedException() {
    RetryConfig config = RetryConfig.custom()
            .maxAttempts(3)                         // up to 3 attempts
            .waitDuration(Duration.ofSeconds(2))    // 2 seconds between attempts
            .build();

    // 2. Create registry and bind metrics
    RetryRegistry retryRegistry = RetryRegistry.of(config);
    MeterRegistry meterRegistry = new SimpleMeterRegistry();
    TaggedRetryMetrics.ofRetryRegistry(retryRegistry).bindTo(meterRegistry);

    // 3. Listen for registry events
    retryRegistry.getEventPublisher()
        .onEntryAdded(entry -> System.out.println("Retry created: " + entry.getAddedEntry().getName()))
        .onEntryRemoved(entry -> System.out.println("Retry removed: " + entry.getRemovedEntry().getName()));

    // 4. Create or retrieve a Retry instance
    Retry retry = retryRegistry.retry("userServiceRetry");

    // 5. Listen for retry lifecycle events
    retry.getEventPublisher()
            .onRetry(event -> System.out.printf("Retry #%d after waiting %d ms%n",
                    event.getNumberOfRetryAttempts(), event.getWaitInterval().toMillis()))
            .onSuccess(event -> System.out.printf("Succeeded after %d attempts%n", 
                    event.getNumberOfRetryAttempts()))
            .onError(event -> System.err.printf("Failed after %d attempts: %s%n", 
                    event.getNumberOfRetryAttempts(), event.getLastThrowable().getMessage()));

    // 6. Prepare the checked supplier for a remote call
    UserQuery query = UserQuery.builder().firstName("Bob").build();
    CheckedSupplier<List<User>> checkedSupplier = () -> userService.searchThrowingException(query);

    // 7. Decorate it with retry logic
    CheckedSupplier<List<User>> retryingSupplier = Retry.decorateCheckedSupplier(retry, checkedSupplier);

    // 8. Execute and handle the result
    try {
        List<User> users = retryingSupplier.get();
        users.forEach(System.out::println);
    } catch (Throwable e) {
        System.err.println("All retries failed: " + e.getMessage());
    }

    // 9. Print out captured metrics
    meterRegistry.forEachMeter(meter -> {
        String desc = meter.getId().getDescription();
        String kind = meter.getId().getTag("kind");

        double count = StreamSupport.stream(meter.measure().spliterator(), false)
                .filter(ms -> ms.getStatistic().name().equals("COUNT"))
                .findFirst()
                .map(Measurement::getValue)
                .orElse(0.0);

        System.out.printf("%s - %s: %.0f%n", desc, kind, count);
    });
}
```

---

- [Home](./../../README.md)
-  [Microservices](./../tutorials.md)