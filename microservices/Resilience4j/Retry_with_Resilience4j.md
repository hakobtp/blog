# Retry with Resilience4j

```info
Author      Ter-Petrosyan Hakob
```

---

In this article, we first give a short introduction to [Resilience4j](https://resilience4j.readme.io/){:target="_blank" rel="noopener"}. 
Then we look closely at its Retry module. You will learn when to use retries, how to set them up, and which features 
[Resilience4j](https://resilience4j.readme.io/){:target="_blank" rel="noopener"}` offers. Along the way, we will also discuss some best practices for retry logic.

## Introduction to Resilience4j

When applications talk to each other over a network, many things can go wrong. A request might time out, 
connections can break, or an upstream service may be down. Sometimes, a sudden load of requests can slow or crash a service.

[Resilience4j](https://resilience4j.readme.io/){:target="_blank" rel="noopener"} is a Java library that helps you make your applications more reliable. 
It gives you tools to handle failures and keep your system running smoothly.

Let’s have a quick look at the modules and their purpose:

Module	Purpose
Retry	Automatically retry a failed remote operation
RateLimiter	Limit how many times we call a remote operation in a certain period
TimeLimiter	Set a time limit when calling remote operation
Circuit Breaker	Fail fast or perform default actions when a remote operation is continuously failing
Bulkhead	Limit the number of concurrent remote operations
Cache	Store results of costly remote operations

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

---

## 📌 Explore More

- 🏠 [Home](./../../README.md)
- 🧩 [Microservices](./../tutorials.md)