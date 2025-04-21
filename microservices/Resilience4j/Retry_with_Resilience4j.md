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

| Module         | Description                                                 |
| -------------- | ----------------------------------------------------------- |
| Retry          | Automatically try a failed call again                       |
| RateLimiter    | Limit how often a call can be made in a given time          |
| TimeLimiter    | Set a maximum time for a call to finish                     |
| Circuit Breaker| Stop calling or use a fallback when failures keep happening |
| Bulkhead       | Limit how many calls can run at the same time               |
| Cache          | Save and reuse results of expensive calls                   |


---

## 📌 Explore More

- 🏠 [Home](./../../README.md)
- 🧩 [Microservices](./../tutorials.md)