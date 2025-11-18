# What Is a Fat JAR and Why Is It Useful?

When you build a Java application, it usually depends on many external libraries — each packaged as a JAR file. 
To run the application, you normally need to have your application’s JAR file plus all its dependency JARs available in the same environment.

This can quickly become messy, especially when deploying the application to another server or a Docker container. 
That’s where a fat JAR (also known as an uber JAR) comes in.

A fat JAR file is a single JAR that contains:
- The application’s own classes and resource files.
-  And all the external JAR files that the application depends on.

In short, it’s a self-contained package — everything your app needs to run is included in one file.

## Why Is It Useful?

Because a fat JAR bundles all dependencies together, you only need to transfer one file to the target environment. 
There’s no need to copy multiple JARs or configure a complex classpath.

Another big advantage is that starting a fat JAR doesn’t require a separate 
Java EE web server such as Apache Tomcat. You can simply run it with a single command:

```bash
java -jar app.jar
```

This makes fat JARs a great choice for containerized environments, especially when deploying with Docker.

If you use Spring Boot, your application can even include an embedded web server (like Tomcat or Jetty). This means that if your Spring Boot app exposes a REST API over HTTP, it can start serving requests immediately — no extra configuration or server installation required.

## In Summary

A fat JAR simplifies deployment by packaging everything your application needs into one file. It’s portable, easy to run, and perfect for Docker-based environments. If you’re building microservices or cloud-native applications, using a fat JAR can save you both time and deployment headaches.

---

- [Home](./../../../README.md)
- [Spring Tutorials](./../../tutorials.md)
- [Spring Core Tutorials](./../core.md)
- [Spring Boot Annotations](./1_Spring_Boot_Annotations.md)