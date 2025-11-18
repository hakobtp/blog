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
|`@PathVariable`|Bind URI template variable|Method param|
|`@RequestParam`|Bind query param/form field|Method param|
|`@RequestBody`|Bind HTTP request body (JSON) to object|Method param|
|`@ResponseBody`|Return JSON/XML response body|Method|
|`@ModelAttribute`|Bind form data to model|Method param|
|`@CrossOrigin`|Allow CORS for APIs|Class or method|

### Validation

|Annotation|Use Case|Used On|
|:-------|:-------|:-------|
|`@Valid`|Trigger validation on DTOs|Method param|
|`@Validated`|Enable validation for method-level constraints|Class, method|
|`@NotNull`, `@Size`, etc.|Validate fields (JSR-303)|DTO field|


### JPA

|Annotation|Use Case|Used On|
|:-------|:-------|:-------|
|`@Entity`|Mark model class as DB entity|Model class|
|`@Table`, `@Column`|Customize DB table/column mappings|Class/field|
|`@Id`, `@GeneratedValue`|Primary key and auto Id generation|Field|
|`@Repository`|Mark DAO for exception handling|Interface/class|
|`@EnableJpaRepositories`|Enable repo scanning|Config class|
|`@Transactional`|Define transactional boundaries|Method/class|
|`@Modifying`, `@Query`|Custom JPA queries or updates|Repository mehtod|

### Scheduling

|Annotation|Use Case|Used On|
|:-------|:-------|:-------|
|`@EnableScheduling`|Enable `@Scheduled` task support|Config class|
|`@Scheduled`|Run mehtod on cron/fixed time|Method|


## Async

|Annotation|Use Case|Used On|
|:-------|:-------|:-------|
|`@EnableAsync`|Enable `@Async` processing|Config class|
|`@Async`|Run mehtod in a separate thread|Method|

### Cashing

|Annotation|Use Case|Used On|
|:-------|:-------|:-------|
|`@EnableCaching`|Enable caching system-wide|Config class|
|`@Cacheable`|Cache result of method|Method|
|`@CachePut`|Update cache with method result|Method|
|`@CacheEvict`|Remove cache entry|Method|

### Conditional 

|Annotation|Use Case|Used On|
|:-------|:-------|:-------|
|`@ConditionalOnProperty`|Load bean based on property valeu|Bean mehtod/class|
|`@ConditionalOnClass`|Load bean if class is on classpath|Bean method/class|

### Security

|Annotation|Use Case|Used On|
|:-------|:-------|:-------|
|`@EnableWebSecurity`|Enable Spring Security|Config class|
|`@PreAuthorize`|Pre-check access with SpEL|Method|
|`@PostAuthorize`|Post-check access with SpEL|Method|
|`@Secured`|Role-based access (basic)|Method|
|`@WithMockUser`|Simulate user in test|Test method|

### Testing

|Annotation|Use Case|Used On|
|:-------|:-------|:-------|
|`@SpringBootTest`|Load full Spring context for intergration testing|Test class|
|`@WebMvcTest`|Test only web layer (controllers)|Test class|
|`@DataJpaTest`|Test only JPA repositories|Test class|
|`@MockBean`|Inject a mock into Spring context|Test field|
|`@TestConfiguration`|Custom config for test only|Inner class|

### Misc

|Annotation|Use Case|Used On|
|:-------|:-------|:-------|
|`@Profile`|Load bean only for specific profile (dev, prod)|Bean|
|`@Scope`|Define bean lifecycle (singleton, prototye)|Bean|
|`@Import`|Import other config classes|Config class|
|`@EnableConfigurtionProperties`|Enable `@ConfigurationProperties` class|Config class|
|`@ConfigurationProperties`|Map grouped properties to a POJO|Class|

---

- [Home](./../../../README.md)
- [Spring Tutorials](./../../tutorials.md)
- [Spring Core Tutorials](./../core.md)
- [HTTP request methods  and Status Codes](./0_HTTP_request_methods.md)
- [What Is a Fat JAR and Why Is It Useful?](./2_What_Is_a_Fat_JAR_and_Why_Is_It_Useful.md)