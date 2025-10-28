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

- [Home](./../../README.md)
- [Spring Tutorials](./../tutorials.md)
