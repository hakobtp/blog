# Spring Boot Annotations

### Core 

|Annotation|Use Case|Used On|
|:-------|:-------|:-------|
|`@SpringBootApplication`|Bootstrap the app with default config, auto-config, component scan|Main class|
|`@ComponentScan`|Scan specific packages for Spring beans|Config/Main class|
|`@Configuration`|Define Spring Beans in Java cofnig|Config class|
|`@Bean`|Declare and customize a bean manually|Method inside config class|
|`@Value`|Inject values from `application.properties`|Field, setter, constructor|
|`@PropertySource`|Load external `.properties` firl|Cofngi class|

### Dependency Injection


---

- [Home](./../../../README.md)
- [Spring Tutorials](./../../tutorials.md)
- [Spring Core Tutorials](./../core.md)