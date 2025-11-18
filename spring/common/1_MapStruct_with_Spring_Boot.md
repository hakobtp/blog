# MapStruct with Spring Boot

When you build applications, you often need to convert between Entities (your database models) and DTOs (Data Transfer Objects).
If you do this manually, you end up writing the same kind of code again and again — it’s boring, easy to break, and hard to maintain.

For example:
```java
UserDto dto = new UserDto();
dto.setName(user.getName());
dto.setEmail(user.getEmail());
dto.setAge(user.getAge());
```

Every time you add or rename a field in User, you must update this code everywhere.
This means more work and a higher chance of mistakes.

Manual mapping causes several issues:
- Too much boilerplate code — you repeat the same logic many times.
- Hard to maintain — if your model changes, you must fix it everywhere.
- Easy to forget fields — you might miss or misname something.

## The Solution: MapStruct

MapStruct is a Java library that solves this problem.
It automatically generates mapping code for you when your project compiles.

This means:
- No reflection at runtime (so it’s very fast)
- Full type safety (mistakes are caught before you run your code)
- Clean, readable, and boilerplate-free mapping

**Here’s a basic example:**
```java
@Mapper
public interface UserMapper {
    UserMapper INSTANCE = Mappers.getMapper(UserMapper.class);
    UserDto toDto(User user);
}
```

When you build your project, MapStruct automatically generates an implementation behind the scenes.
You don’t write it — but it’s as fast as if you did.

MapStruct doesn’t use reflection.
Instead, it generates plain Java code that looks just like code you would write yourself.

For example, the generated class might look like this:
```java
@Generated
public class UserMapperImpl implements UserMapper {

    @Override
    public UserDto toDto(User user) {
        if (user == null) return null;
        UserDto dto = new UserDto();
        dto.setName(user.getName());
        dto.setEmail(user.getEmail());
        dto.setAge(user.getAge());
        return dto;
    }
}
```

You get the same performance as manual mapping — but with none of the repetitive work.

## Custom Field Mapping

Sometimes the field names in your Entity and DTO don’t match.
MapStruct lets you fix this easily using the `@Mapping` annotation.

```java
@Mappings({
    @Mapping(source = "emailAddress", target = "email"),
    @Mapping(source = "birthDate", target = "dateOfBirth")
})
UserDto toDto(User user);
```

Here:

- `source` is the field name in your entity
- `target` is the field name in your DTO

This is much cleaner than doing it manually.
It’s also type-safe, so if one of the fields changes, the compiler will warn you.

## Bidirectional Mapping

You often need to map objects both ways — from Entity to DTO and back again.
MapStruct can do this with almost no extra code.

```java
@Mapper
public interface OrderMapper {
    @Mapping(source = "customer.name", target = "customerName")
    OrderDto toDto(Order order);

    @InheritInverseConfiguration
    Order toEntity(OrderDto dto);
}
```

The `@InheritInverseConfiguration` annotation tells MapStruct to reuse the same rules in reverse.
One mapper, both directions, zero duplication.

## Nested Mapping

MapStruct can also map nested objects — even several layers deep.

```java
@Mapper
public interface OrderMapper {
    @Mapping(source = "customer.address.city", target = "customerCity")
    @Mapping(source = "customer.address.zipCode", target = "postalCode")
    OrderDto toDto(Order order);
}
```

It automatically follows the nested path (like `customer.address.city`) and safely copies only what you need.
You don’t need extra helper methods or null checks.

This is great for complex domain models — your code stays clean and easy to read.

## Integration with Spring Boot

MapStruct integrates perfectly with Spring.
You can make your mappers Spring beans by setting `componentModel = "spring"`.

Maven Configuration Example

Here’s how your Maven `pom.xml` might look:

```xml
...

<properties>
    <java.version>17</java.version>
    <org.mapstruct.version>1.6.3</org.mapstruct.version>
    <lombok-mapstruct-binding.version>0.2.0</lombok-mapstruct-binding.version>
</properties>

...

<dependencies>
    ... 
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.mapstruct</groupId>
        <artifactId>mapstruct</artifactId>
        <version>${org.mapstruct.version}</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <annotationProcessorPaths>
                    <path>
                        <groupId>org.mapstruct</groupId>
                        <artifactId>mapstruct-processor</artifactId>
                        <version>${org.mapstruct.version}</version>
                    </path>
                    <path>
                        <groupId>org.projectlombok</groupId>
                        <artifactId>lombok</artifactId>
                    </path>
                    <path>
                        <groupId>org.projectlombok</groupId>
                        <artifactId>lombok-mapstruct-binding</artifactId>
                        <version>${lombok-mapstruct-binding.version}</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>

```

Then create your mapper like this:

```java
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserDto toDto(User user);
}
```

Now you can inject it directly into your Spring services:

```java
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserMapper userMapper;
}
```

MapStruct automatically registers the mapper as a Spring bean — no manual setup needed.

## Custom Mapping Logic

Sometimes, you need custom transformation logic.
For example, combining two fields into one:

```java
@Mapper(componentModel = "spring")
public interface UserMapper {
    @Mapping(target = "fullName", expression = "java(user.getFirstName() + ' ' + user.getLastName())")
    UserDto toDto(User user);
}
```

You can use Java expressions inside `@Mapping` to compute values during mapping.
This keeps your transformation logic inside the mapper instead of spreading it through your service layer.

## Mapping Collections

MapStruct automatically maps collections like `List`, `Set`, or even `Map`.

```java
@Mapper(componentModel = "spring")
public interface ProductMapper {
    ProductDto toDto(Product product);
    List<ProductDto> toDtoList(List<Product> products);
}
```

It applies the same mapping rules to every element.
You don’t need to loop manually — MapStruct does it for you.

## Updating Existing Entities

You can also update existing entities instead of creating new ones.

```java
@Mapper(componentModel = "spring")
public interface UserMapper {
    void updateUserFromDto(UserDto dto, @MappingTarget User entity);
}
```

To ignore null fields during updates, use:

```java
@Mapper(componentModel = "spring")
@BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)
public interface UserMapper {
    void updateUserFromDto(UserDto dto, @MappingTarget User entity);
}
```

This keeps existing values untouched if the incoming DTO has `null` properties.

## Using @Named and Qualified Mappings

Sometimes, MapStruct has more than one possible method to perform a conversion.
In such cases, you can tell MapStruct exactly which method to use with the `@Named` annotation.

Let’s imagine you are building an e-commerce system.
You want to convert a numeric `category ID` into a `CategoryDto`, using an enum or a lookup.

```java
@Named("categoryIdToCategoryDto")
default CategoryDto categoryIdToCategoryDto(Long categoryId) {
    return CategoryDto.fromEnum(CategoryEnum.getById(categoryId));
}
```

This method converts a `Long` (the category ID) into a CategoryDto.
By marking it with` @Named("categoryIdToCategoryDto")`, you can reuse it in other mappers that need this specific conversion.

Now, let’s see how we can use this method inside another mapper.

```java
@Mapper(
    componentModel = "spring",
    uses = {CategoryMapper.class, PriceMapper.class}
)
public interface ProductMapper {

    @Mapping(source = "id", target = "productId")
    @Mapping(source = "name", target = "productName")
    @Mapping(source = "categoryId", target = "category", qualifiedByName = "categoryIdToCategoryDto")
    @Mapping(source = "priceInCents", target = "price", qualifiedByName = "centsToPriceDto")
    ProductResponseDto toResponseDto(Product product);
}
```

Explanation

- `uses = {...}` — tells MapStruct that this mapper can use methods from the listed mappers (`CategoryMapper`, `PriceMapper`, etc.).
- `qualifiedByName` — connects a target field to a specific helper method marked with `@Named`.

This approach makes your mappings more explicit and easier to control, especially when you have several possible converters for the same types.

Use `@Named + qualifiedByName` when you want precise control over which helper method MapStruct uses for a particular field.


## Understanding the uses Property

When building applications, it’s common to have objects that contain other complex objects.
For example, a `Product` may have a `Category` and a `Price`.
If each of these needs its own mapping logic, we can organize them into separate mappers.
MapStruct’s uses property helps us do exactly that.

Let’s imagine you have three classes:

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Product {
    private Long id;
    private String name;
    private Long categoryId;
    private Long priceInCents;
}

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class CategoryDto {
    private Long id;
    private String name;
}

@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class PriceDto {
    private BigDecimal amount;
    private String currency;
}
```

You want to map a `Product` into a `ProductResponseDto` that looks like this:

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class ProductResponseDto {
    private Long productId;
    private String productName;
    private CategoryDto category;
    private PriceDto price;
}
```

Clearly, the conversion from `categoryId → CategoryDto` and from `priceInCents → PriceDto` should be handled separately.
That’s where helper mappers come in.

### Step 1: Create Helper Mappers

CategoryMapper:
```java
@Mapper(componentModel = "spring")
public interface CategoryMapper {

    @Named("categoryIdToCategoryDto")
    default CategoryDto categoryIdToCategoryDto(Long id) {
        // Simulated enum or lookup
        return new CategoryDto(id, switch (id.intValue()) {
            case 1 -> "Electronics";
            case 2 -> "Books";
            case 3 -> "Home & Kitchen";
            default -> "Other";
        });
    }
}
```

PriceMapper:
```java
@Mapper(componentModel = "spring")
public interface PriceMapper {

    @Named("centsToPriceDto")
    default PriceDto centsToPriceDto(Long cents) {
        if (cents == null) return null;
        BigDecimal amount = BigDecimal.valueOf(cents, 2);
        return new PriceDto(amount, "USD");
    }
}
```

Each of these mappers contains a method marked with `@Named`, so that we can refer to them in other mappers.

## Step 2: Use Them in the Main Mapper

Now we’ll create our main mapper, `ProductMapper`, and connect the helper mappers with the uses property.

```java
@Mapper(
    componentModel = "spring",
    uses = {CategoryMapper.class, PriceMapper.class}
)
public interface ProductMapper {

    @Mapping(source = "id", target = "productId")
    @Mapping(source = "name", target = "productName")
    @Mapping(source = "categoryId", target = "category", qualifiedByName = "categoryIdToCategoryDto")
    @Mapping(source = "priceInCents", target = "price", qualifiedByName = "centsToPriceDto")
    ProductResponseDto toResponseDto(Product product);
}
```

Explanation

- `uses = {...}` — tells MapStruct which other mappers this mapper can use.
- `qualifiedByName` — specifies exactly which `@Named` method from those mappers should be used.

This design keeps the code modular and easy to maintain.
Each helper mapper does one thing, and the main mapper just combines them.

The Generated Code:
```java
@Component
public class ProductMapperImpl implements ProductMapper {

    private final CategoryMapper categoryMapper;
    private final PriceMapper priceMapper;

    public ProductMapperImpl(CategoryMapper categoryMapper, PriceMapper priceMapper) {
        this.categoryMapper = categoryMapper;
        this.priceMapper = priceMapper;
    }

    @Override
    public ProductResponseDto toResponseDto(Product product) {
        if (product == null) {
            return null;
        }

        ProductResponseDto dto = new ProductResponseDto();

        dto.setProductId(product.getId());
        dto.setProductName(product.getName());
        dto.setCategory(categoryMapper.categoryIdToCategoryDto(product.getCategoryId()));
        dto.setPrice(priceMapper.centsToPriceDto(product.getPriceInCents()));

        return dto;
    }
}
```

**Note** — When Working With Entity Relationships

If your application uses entities that have relationships with other entities, such as:

```java
class Product {
    private Long id;
    private String name;
    private Category category; // Many-to-One relationship
}
```

you can create separate mappers for both the parent and the child entities.
For example:

- A `CategoryMapper` for converting between Category and CategoryDto
- A `ProductMapper` for converting between Product and ProductDto

Then, your `ProductMapper` can use the `CategoryMapper` through the uses property:

```java
@Mapper(componentModel = "spring", uses = {CategoryMapper.class})
public interface ProductMapper {
    @Mapping(source = "category", target = "categoryDto")
    ProductDto toDto(Product product);
}
```

MapStruct will automatically inject and call the child mapper (`CategoryMapper`) when mapping the relationship field.
This keeps your mappers clean, modular, and consistent, especially in larger projects with many entity relationships.

**Example:** Bidirectional Relationship

```java
public class Category {
    private Long id;
    private String name;
    private List<Product> products; // One-to-Many
}

public class Product {
    private Long id;
    private String name;
    private Category category; // Many-to-One
}
```

**And two DTOs:**

```java
public class CategoryDto {
    private Long id;
    private String name;
    private List<ProductDto> products;
}

public class ProductDto {
    private Long id;
    private String name;
    private CategoryDto category;
}
```

**Create a ProductMapper:**

```java
@Mapper(componentModel = "spring", uses = {CategoryMapper.class})
public interface ProductMapper {

    @Mapping(source = "category", target = "category")
    ProductDto toDto(Product product);

    @Mapping(source = "category", target = "category")
    Product toEntity(ProductDto productDto);
}
```
Here, the `ProductMapper` `uses` the `CategoryMapper` to convert the category field.

**Create a CategoryMapper**

Now, the tricky part — Category also references Product.
If we mapped both sides directly, we would get infinite recursion (because CategoryDto contains ProductDto, which again contains CategoryDto, and so on).

To solve this, we can ignore one direction of the mapping

```java
@Mapper(componentModel = "spring", uses = {ProductMapper.class})
public interface CategoryMapper {

    @Mapping(target = "products", qualifiedByName = "mapProductsWithoutCategory")
    CategoryDto toDto(Category category);

    @Mapping(target = "products", ignore = true)
    Category toEntity(CategoryDto categoryDto);

    // Custom method to map products without back-reference to category
    @Named("mapProductsWithoutCategory")
    default List<ProductDto> mapProductsWithoutCategory(List<Product> products) {
        if (products == null) return null;
        return products.stream()
                .map(p -> new ProductDto(p.getId(), p.getName(), null)) // Avoid recursion
                .toList();
    }
}

```

## Using Abstract Classes with Spring Beans

Sometimes, your mapping logic needs to use a Spring service or component.
You can’t do that in a normal interface, but you can use an abstract class instead.

Here’s an example:

```java
@Mapper(componentModel = "spring")
public abstract class OrderMapper {

    @Autowired
    private PriceService priceService;

    @Mapping(target = "totalPrice", expression = "java(priceService.calculateTotal(order))")
    public abstract OrderDto toDto(Order order);
}
```

What’s happening here

- The mapper is defined as an abstract class, not an interface.
- Spring automatically injects the `PriceService` bean.
- The mapping uses an expression to call a method inside the injected service.

So you can mix generated mapping logic with custom, service-based transformations — very useful when part of your mapping depends on external data or business logic.

## Key Properties of @Mapping

The `@Mapping` annotation tells MapStruct how one field in the source object should be mapped to a field in the target object.

Let’s look at the most useful properties in simple terms.

| Property                             | Description                                                                 | Example                                                                                                          |
| ------------------------------------ | --------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| **target**                           | The name of the target field (the one you want to set).                     | `@Mapping(target = "email", source = "emailAddress")`                                                            |
| **source**                           | The name of the source field (the one you read from).                       | `@Mapping(source = "user.name", target = "username")`                                                            |
| **expression**                       | Custom Java code used to calculate a value.                                 | `@Mapping(target = "age", expression = "java(Period.between(user.getBirthDate(), LocalDate.now()).getYears())")` |
| **constant**                         | Sets a fixed value.                                                         | `@Mapping(target = "status", constant = "ACTIVE")`                                                               |
| **defaultValue**                     | Used when the source is `null`.                                             | `@Mapping(target = "country", defaultValue = "Unknown")`                                                         |
| **defaultExpression**                | Runs a Java expression only if the source is `null`.                        | `@Mapping(target = "createdAt", defaultExpression = "java(LocalDate.now())")`                                    |
| **ignore**                           | Tells MapStruct to skip this field.                                         | `@Mapping(target = "password", ignore = true)`                                                                   |
| **qualifiedByName**                  | Links a field to a method marked with `@Named`.                             | `@Mapping(source = "statusId", target = "status", qualifiedByName = "idToStatusDto")`                            |
| **dateFormat**                       | Formats date conversions.                                                   | `@Mapping(source = "createdAt", target = "createdDate", dateFormat = "yyyy-MM-dd")`                              |
| **numberFormat**                     | Formats numbers when converting between `String` and `Number`.              | `@Mapping(source = "price", target = "priceText", numberFormat = "#,##0.00")`                                    |
| **nullValueCheckStrategy**           | Controls if null checks are added.                                          | Default is `ON_IMPLICIT_CONVERSION`.                                                                             |
| **nullValuePropertyMappingStrategy** | Defines what happens when a source property is null.                        | `SET_TO_NULL` (default) or `IGNORE`.                                                                             |
| **dependsOn**                        | Ensures one field is mapped before another (used for dependent properties). | `@Mapping(target = "fullName", dependsOn = "firstName")`                                                         |


## Example — Combining Everything

Here’s a mapper that shows many of these features in one place:

```java
@Mapper(componentModel = "spring", uses = {UserHelperMapper.class})
public abstract class UserMapper {

    @Autowired
    protected TimeService timeService;

    @Mapping(source = "id", target = "userId")
    @Mapping(source = "emailAddress", target = "email")
    @Mapping(target = "status", constant = "ACTIVE")
    @Mapping(target = "age", expression = "java(timeService.calculateAge(user.getBirthDate()))")
    @Mapping(target = "createdAt", defaultExpression = "java(LocalDate.now())")
    @Mapping(target = "role", qualifiedByName = "roleIdToRoleDto")
    @Mapping(target = "temporaryField", ignore = true)
    public abstract UserDto toDto(User user);
}
```

Explanation:

- `constant = "ACTIVE`" → sets a fixed value.
- `expression` → calls a Spring bean (timeService) to calculate something.
- `defaultExpression` → runs only when the source field is null.
- `qualifiedByName` → uses another method with a matching @Named name.
- `ignore` → skips a field completely.


MapStruct’s `@Mapping` annotation is extremely flexible — it can handle almost any field conversion you need.
The key is to use the right combination of:

- `@Named` + `qualifiedByName` for specialized mappings,
- `expression` or `defaultExpression` for dynamic logic,
- and `abstract mappers` if you need to inject Spring services.

## Advantages of MapStruct

| Feature               | Description                                      |
| --------------------- | ------------------------------------------------ |
| **Type-safe mapping** | All mappings are checked at compile time.        |
| **No reflection**     | Runs fast because everything is plain Java code. |
| **Easy maintenance**  | No repeated or duplicated code.                  |
| **Spring-friendly**   | Works smoothly with Lombok and Spring Boot.      |
| **Clean output**      | Generates readable, production-quality classes.  |


## Conclusion

MapStruct gives you the speed of automatic mapping and the safety of handwritten code.
It removes the boring part of converting between DTOs and Entities while keeping everything clean, fast, and reliable.

If you’re using Spring Boot and Lombok, MapStruct fits right in — and once you start using it, you’ll never want to go back to manual mapping again.

---

- [Home](./../../README.md)
- [Spring Tutorials](./../tutorials.md)
