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

|Annotation|Use Case|Used On|
|:-------|:-------|:-------|
|`@Autowired`|Auto-inject dependencies by type|Field, setter, constructor|
|`@Qualifier`|Specify which bean to inject if multiple|Field, param|
|`@Primary`|Mark default bean when multiple types exist|Bean class or method|
|`@Inject`|Java standard alternative to `@Autowired`|Field|
|`@Resource`|Inject bean by name|Field|


### Stereotype

|Annotation|Use Case|Used On|
|:-------|:-------|:-------|
|`@Component`|General-purpose Spring-managed bean|Class|
|`@Service`|Business logic layer bean|Class|
|`@Repository`|DAO class with exception translation|Class or interface|
|`@Controller`|MVC controller returning views|Class|
|`@RestController`|REST controller returning JSON/XML|Class|

### Web (MVC)

|Annotation|Use Case|Used On|
|:-------|:-------|:-------|
|`@RequestMapping`|Map HTTP request to methods|Class/method|
|`@GetMapping`, `@PostMapping`, etc.|Shorthand for request methods|Method|
|`@`|||
|`@`|||
|`@`|||
|`@`|||
|`@`|||
|`@`|||

---

- [Home](./../../../README.md)
- [Spring Tutorials](./../../tutorials.md)
- [Spring Core Tutorials](./../core.md)