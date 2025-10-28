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







---

When mapping DTOs and Entities manually, we end up writing
repetitive, error-prone code. Every time a field changes, we must update it in multiple places.

```java
UserDto dto = new UserDto();
dto.setName(user.getName());
dto.setEmail(user.getEmail());
dto.setAge(user.getAge());
```

Potential issues with this code:
- Too much boilerplate
- Hard to maintain when models evolve
- Risk of missing or mismatched fields

## The Solution: MapStruct

MapStruct automatically generates mapping code at compile time,
eliminating repetitive and error-prone manual work. It avoids reflection,
runs lightning-fast, ensures full type safety, and keeps your code clean -
no boilerplate required.

```java
@Mapper
public interface UserMapper {
    UserMapper INSTANSE = Mappers.getMapper(UserMapper.class);
    UserDto toDto(User user);
}
```

**Result:** MapStruct generates the implementation behind the scenes and
you get a ready-to-use mapper without writing a single line of boilerplate.


### Under the Hood

Why MapStruct Is Faster?

- MapStruct generates plain Java code, no reflection at runtime.
- Mapping happens just like you would write it manually, but generated automatically.
- Compared to ModelMapper, there’s zero runtime cost and full compile-time validation.

What MapStruct Generates:
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

**Result:** performance like manual mapping, but maintainability like magic.

### Custom Field Mapping

When our field names don’t match, MapStruct will let us define custom
mappings easily using the `@Mapping` annotation.

```java
@Mappings({
    @Mapping(source = "emailAddress", target = "email"),
    @Mapping(source = "birthDate", target = "dateOfBirth")
})
UserDto toDto(User user);
```

The source field comes from the entity, and target is the DTO field.
This is very clean, type-safe, and easy to maintain - even as our models
evolve or grow in complexity. MapStruct ensures every field is mapped
explicitly, reducing human error and keeping our codebase consistent
over time.

### Bidirectional Mapping

MapStruct supports bidirectional object mapping and conversions,
making it easy to handle complex models.

```java
@Mapper
public interface OrderMapping {
    @Mapping(source = "customer.name", target = "customerName")
    OrderDto toDto(Order order);

    @InheritInverseConfiguration
    Order toEntity(OrderDto dto);
}
```

- `@InheritInverseConfiguration` automatically reuses the same mapping rules in the opposite direction.

One mapper - both directions, zero duplication. It is perfect for mapping
entities with nested relationships (like Order → Customer → Address)
and DTOs that are used in both read and write operations.

### Nested Mapping

MapStruct makes it easy to map nested objects and you can directly
access inner fields without manual null checks or helper methods.

```java
@Mapper
public interface OrderMapping {

    @Mapping(source = "customer.address.sity", target = "customerCity")
    @Mapping(source = "customer.address.zipCode", target = "postalCode")
    OrderDto toDto(Order order);

}    
```

It automatically navigates through nested objects and maps only the
needed fields.

This approach is ideal for complex domain models, MapStruct eliminates
deep boilerplate mapping code and significantly improves readability
and maintainability.

## Integration with Spring

MapStruct easily integrates with the Spring ecosystem, allowing you to
inject mappers like any other Spring bean.

project maven pom

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.5.6</version>
        <relativePath/>
    </parent>
    ...

    <properties>
        <java.version>17</java.version>
        <org.mapstruct.version>1.6.3</org.mapstruct.version>
        <lombok-mapstruct-binding.version>0.2.0</lombok-mapstruct-binding.version>
    </properties>
    <dependencies>
        ....

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
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

```java
@Mapper(componentModel = "spring")
public interface UserMapper {
    UserDto toDto(User user);
}
```

```java
@Service
@RequiredArgsConstructor
public class UserService {
    private final UserMapper userMapper;
}
```

By setting `componentModel = "spring"`, MapStruct automatically registers the mapper as a Spring bean.

No manual initialization required. MapStruct works seamlessly with Spring’s
dependency injection, keeping your architecture clean, modular, and easy to
test.

### Custom Mapping

MapStruct allows defining custom logic when default field mapping isn’t
enough.
You can write custom expressions or delegate the logic to another
method.

```java
@Mapper(componentModel = "spring")
public interface UserMapper {
 
    @Mapping(target = "fullName", expression = "java(user.getFirstName() + ' ' + user.getLastName())")
    UserDto toDto(User user);
}
```

Custom expressions give full control when mapping requires transformations
or computed fields.
It is flexible for complex conversions while keeping all transformation logic
centralized inside the mapper, preventing unnecessary clutter in service
layers.

## Mapping Collections

MapStruct automatically maps collections between objects - no loops or
manual conversion needed. It handles List, Set, and even Map types out
of the box.

```java
@Mapper(componentModel = "spring")
public interface ProductMapper {
    ProductDto toDto(Product product);
    List<ProductDto> toDtoList(List<Product> products);
}    
```

Each element is mapped using the same mapping rules as for single
objects.

It automatically maps each element, supports nested mappers for
complex types, and greatly simplifies batch data transformations.

## Updating Existing Entities

MapStruct allows updating existing entities without creating new objects - ideal for partial updates or PATCH operations.

```java
@Mapper(componentModel = "spring")
public interface UserMapper {
 
    updateUserFromDto(UserDto dto, @MappingTarget User entity);
}
```

 @BeanMapping(nullValuePropertyMappingStrategy = NullValuePropertyMappingStrategy.IGNORE)


Instead of returning a new User, this method updates the existing
instance directly - keeping unchanged fields intact.

It is perfect for PATCH endpoints - it prevents data loss in persistent
entities and keeps the service logic clean and focused.

# some title 

some text

 @Named("approvalStatusToApprovalStatusDto")
    default ApprovalStatusDto approvalStatusToApprovalStatusDto(Long approvalStatusId) {
        return ApprovalStatusDto.buildFromEnum(ApprovalStatusEnum.getById(approvalStatusId));
    }

    @Mapper(componentModel = "spring", uses = {ChangeRequestInfoMapper.class, ChangeRequestInfoMapperWithSubmitted.class})
public interface TraderGroupMapper {

    @Mapping(source = "groupId", target = "traderGroupGroupId")
    @Mapping(source = "protocolId", target = "protocol", qualifiedByName = "protocolToProtocolDto")
    @Mapping(source = "approvalStatusId", target = "approvalStatus", qualifiedByName = "approvalStatusToApprovalStatusDto")
    TraderGroupExternalResponseDto toExternalResponseDto(TraderGroupDto traderGroupDto);

# some title
example with absract class werwe we can inject spring bean


## some title

write example and explian all proprties 


@Repeatable(Mappings.class)
@Retention(RetentionPolicy.CLASS)
@Target({ ElementType.METHOD, ElementType.ANNOTATION_TYPE })
public @interface Mapping {

    /**
     * The target name of the configured property as defined by the JavaBeans specification. The same target property
     * must not be mapped more than once.
     * <p>
     * If used to map an enum constant, the name of the constant member is to be given. In this case, several values
     * from the source enum may be mapped to the same value of the target enum.
     *
     * @return The target name of the configured property or enum constant
     */
    String target();

    /**
     * The source to use for this mapping. This can either be:
     * <ol>
     * <li>The source name of the configured property as defined by the JavaBeans specification.
     * <p>
     * This may either be a simple property name (e.g. "address") or a dot-separated property path (e.g. "address.city"
     * or "address.city.name"). In case the annotated method has several source parameters, the property name must
     * qualified with the parameter name, e.g. "addressParam.city".</li>
     * <li>When no matching property is found, MapStruct looks for a matching parameter name instead.</li>
     * <li>When used to map an enum constant, the name of the constant member is to be given.</li>
     * </ol>
     * This attribute can not be used together with {@link #constant()} or {@link #expression()}.
     *
     * @return The source name of the configured property or enum constant.
     */
    String source() default "";

    /**
     * A format string as processable by {@link SimpleDateFormat} if the attribute is mapped from {@code String} to
     * {@link Date} or vice-versa. Will be ignored for all other attribute types and when mapping enum constants.
     *
     * @return A date format string as processable by {@link SimpleDateFormat}.
     */
    String dateFormat() default "";

    /**
     * A format string as processable by {@link DecimalFormat} if the annotated method maps from a
     *  {@link Number} to a {@link String} or vice-versa. Will be ignored for all other element types.
     *
     * @return A decimal format string as processable by {@link DecimalFormat}.
     */
    String numberFormat() default "";

    /**
     * A constant {@link String} based on which the specified target property is to be set.
     * <p>
     * When the designated target property is of type:
     * </p>
     * <ol>
     * <li>primitive or boxed (e.g. {@code java.lang.Long}).
     * <p>
     * MapStruct checks whether the primitive can be assigned as valid literal to the primitive or boxed type.
     * </p>
     * <ul>
     * <li>
     * If possible, MapStruct assigns as literal.
     * </li>
     * <li>
     * If not possible, MapStruct will try to apply a user defined mapping method.
     * </li>
     * </ul>
     * </li>
     * <li>other
     * <p>
     * MapStruct handles the constant as {@code String}. The value will be converted by applying a matching method,
     * type conversion method or built-in conversion.
     * <p>
     * </li>
     * </ol>
     * <p>
     * You can use {@link #qualifiedBy()} or {@link #qualifiedByName()} to force the use of a conversion method
     * even when one would not apply. (e.g. {@code String} to {@code String})
     * </p>
     * <p>
     * This attribute can not be used together with {@link #source()}, {@link #defaultValue()},
     * {@link #defaultExpression()} or {@link #expression()}.
     *
     * @return A constant {@code String} constant specifying the value for the designated target property
     */
    String constant() default "";

    /**
     * An expression {@link String} based on which the specified target property is to be set.
     * <p>
     * Currently, Java is the only supported "expression language" and expressions must be given in form of Java
     * expressions using the following format: {@code java(<EXPRESSION>)}. For instance the mapping:
     * <pre><code>
     * &#64;Mapping(
     *     target = "someProp",
     *     expression = "java(new TimeAndFormat( s.getTime(), s.getFormat() ))"
     * )
     * </code></pre>
     * <p>
     * will cause the following target property assignment to be generated:
     * <p>
     * {@code targetBean.setSomeProp( new TimeAndFormat( s.getTime(), s.getFormat() ) )}.
     * <p>
     * Any types referenced in expressions must be given via their fully-qualified name. Alternatively, types can be
     * imported via {@link Mapper#imports()}.
     * <p>
     * This attribute can not be used together with {@link #source()}, {@link #defaultValue()},
     * {@link #defaultExpression()}, {@link #qualifiedBy()}, {@link #qualifiedByName()} or {@link #constant()}.
     *
     * @return An expression specifying the value for the designated target property
     */
    String expression() default "";

    /**
     * A defaultExpression {@link String} based on which the specified target property is to be set
     * if and only if the specified source property is null.
     * <p>
     * Currently, Java is the only supported "expression language" and expressions must be given in form of Java
     * expressions using the following format: {@code java(<EXPRESSION>)}. For instance the mapping:
     * <pre><code>
     * &#64;Mapping(
     *     target = "someProp",
     *     defaultExpression = "java(new TimeAndFormat( s.getTime(), s.getFormat() ))"
     * )
     * </code></pre>
     * <p>
     * will cause the following target property assignment to be generated:
     * <p>
     * {@code targetBean.setSomeProp( new TimeAndFormat( s.getTime(), s.getFormat() ) )}.
     * <p>
     * Any types referenced in expressions must be given via their fully-qualified name. Alternatively, types can be
     * imported via {@link Mapper#imports()}.
     * <p>
     * This attribute can not be used together with {@link #expression()}, {@link #defaultValue()}
     * or {@link #constant()}.
     *
     * @return An expression specifying a defaultValue for the designated target property if the designated source
     * property is null
     *
     * @since 1.3
     */
    String defaultExpression() default "";

    /**
     * Whether the property specified via {@link #target()} should be ignored by the generated mapping method or not.
     * This can be useful when certain attributes should not be propagated from source or target or when properties in
     * the target object are populated using a decorator and thus would be reported as unmapped target property by
     * default.
     *
     * @return {@code true} if the given property should be ignored, {@code false} otherwise
     */
    boolean ignore() default false;

    /**
     * A qualifier can be specified to aid the selection process of a suitable mapper. This is useful in case multiple
     * mapping methods (hand written or generated) qualify and thus would result in an 'Ambiguous mapping methods found'
     * error. A qualifier is a custom annotation and can be placed on a hand written mapper class or a method.
     * <p>
     * Note that {@link #defaultValue()} usage will also be converted using this qualifier.
     *
     * @return the qualifiers
     * @see Qualifier
     */
    Class<? extends Annotation>[] qualifiedBy() default { };

    /**
     * String-based form of qualifiers; When looking for a suitable mapping method for a given property, MapStruct will
     * only consider those methods carrying directly or indirectly (i.e. on the class-level) a {@link Named} annotation
     * for each of the specified qualifier names.
     * <p>
     * Note that annotation-based qualifiers are generally preferable as they allow more easily to find references and
     * are safe for refactorings, but name-based qualifiers can be a less verbose alternative when requiring a large
     * number of qualifiers as no custom annotation types are needed.
     * <p>
     * Note that {@link #defaultValue()} usage will also be converted using this qualifier.
     *
     * @return One or more qualifier name(s)
     * @see #qualifiedBy()
     * @see Named
     */
    String[] qualifiedByName() default { };

    /**
     * A qualifier can be specified to aid the selection process of a suitable presence check method.
     * This is useful in case multiple presence check methods qualify and thus would result in an
     * 'Ambiguous presence check methods found' error.
     * A qualifier is a custom annotation and can be placed on a hand written mapper class or a method.
     * This is similar to the {@link #qualifiedBy()}, but it is only applied for {@link Condition} methods.
     *
     * @return the qualifiers
     * @see Qualifier
     * @see #qualifiedBy()
     * @since 1.5
     */
    Class<? extends Annotation>[] conditionQualifiedBy() default { };

    /**
     * String-based form of qualifiers for condition / presence check methods;
     * When looking for a suitable presence check method for a given property, MapStruct will
     * only consider those methods carrying directly or indirectly (i.e. on the class-level) a {@link Named} annotation
     * for each of the specified qualifier names.
     *
     * This is similar like {@link #qualifiedByName()} but it is only applied for {@link Condition} methods.
     * <p>
     *   Note that annotation-based qualifiers are generally preferable as they allow more easily to find references and
     *   are safe for refactorings, but name-based qualifiers can be a less verbose alternative when requiring a large
     *   number of qualifiers as no custom annotation types are needed.
     * </p>
     *
     *
     * @return One or more qualifier name(s)
     * @see #conditionQualifiedBy()
     * @see #qualifiedByName()
     * @see Named
     * @since 1.5
     */
    String[] conditionQualifiedByName() default { };

    /**
     * A conditionExpression {@link String} based on which the specified property is to be checked
     * whether it is present or not.
     * <p>
     * Currently, Java is the only supported "expression language" and expressions must be given in form of Java
     * expressions using the following format: {@code java(<EXPRESSION>)}. For instance the mapping:
     * <pre><code>
     * &#64;Mapping(
     *     target = "someProp",
     *     conditionExpression = "java(s.getAge() &#60; 18)"
     * )
     * </code></pre>
     * <p>
     * will cause the following target property assignment to be generated:
     * <pre><code>
     *     if (s.getAge() &#60; 18) {
     *         targetBean.setSomeProp( s.getSomeProp() );
     *     }
     * </code></pre>
     * <p>
     * Any types referenced in expressions must be given via their fully-qualified name. Alternatively, types can be
     * imported via {@link Mapper#imports()}.
     * <p>
     * This attribute can not be used together with {@link #expression()} or {@link #constant()}.
     *
     * @return An expression specifying a condition check for the designated property
     *
     * @since 1.5
     */
    String conditionExpression() default "";

    /**
     * Specifies the result type of the mapping method to be used in case multiple mapping methods qualify.
     *
     * @return the resultType to select
     */
    Class<?> resultType() default void.class;

    /**
     * One or more properties of the result type on which the mapped property depends. The generated method
     * implementation will invoke the setters of the result type ordered so that the given dependency relationship(s)
     * are satisfied. Useful in case one property setter depends on the state of another property of the result type.
     * <p>
     * An error will be raised in case a cycle in the dependency relationships is detected.
     *
     * @return the dependencies of the mapped property
     */
    String[] dependsOn() default { };

    /**
     * In case the source property is {@code null}, the provided default {@link String} value is set.
     * <p>
     * When the designated target property is of type:
     * </p>
     * <ol>
     * <li>primitive or boxed (e.g. {@code java.lang.Long}).
     * <p>
     * MapStruct checks whether the primitive can be assigned as valid literal to the primitive or boxed type.
     * </p>
     * <ul>
     * <li>
     * If possible, MapStruct assigns as literal.
     * </li>
     * <li>
     * If not possible, MapStruct will try to apply a user defined mapping method.
     * </li>
     * </ul>
     * <p>
     * </li>
     * <li>other
     * <p>
     * MapStruct handles the constant as {@code String}. The value will be converted by applying a matching method,
     * type conversion method or built-in conversion.
     * <p>
     * </li>
     * </ol>
     * <p>
     * This attribute can not be used together with {@link #constant()}, {@link #expression()}
     * or {@link #defaultExpression()}.
     *
     * @return Default value to set in case the source property is {@code null}.
     */
    String defaultValue() default "";

    /**
     * Determines when to include a null check on the source property value of a bean mapping.
     *
     * Can be overridden by the one on {@link MapperConfig}, {@link Mapper} or {@link BeanMapping}.
     *
     * @since 1.3
     *
     * @return strategy how to do null checking
     */
    NullValueCheckStrategy nullValueCheckStrategy() default ON_IMPLICIT_CONVERSION;

    /**
     * The strategy to be applied when the source property is {@code null} or not present. If no strategy is configured,
     * the strategy given via {@link MapperConfig#nullValuePropertyMappingStrategy()},
     * {@link BeanMapping#nullValuePropertyMappingStrategy()} or
     * {@link Mapper#nullValuePropertyMappingStrategy()} will be applied.
     *
     * {@link NullValuePropertyMappingStrategy#SET_TO_NULL} will be used by default.
     *
     * @since 1.3
     *
     * @return The strategy to be applied when {@code null} is passed as source property value or the source property
     * is not present.
     */
    NullValuePropertyMappingStrategy nullValuePropertyMappingStrategy()
        default NullValuePropertyMappingStrategy.SET_TO_NULL;

    /**
     * Allows detailed control over the mapping process.
     *
     * @return the mapping control
     *
     * @since 1.4
     *
     * @see org.mapstruct.control.DeepClone
     * @see org.mapstruct.control.NoComplexMapping
     * @see org.mapstruct.control.MappingControl
     */
    Class<? extends Annotation> mappingControl() default MappingControl.class;

}


# Advantages

FEATURE DESCRIPTION
Type-safe mapping | All conversions are validated at compile time
Zero reflection | Ensures excellent runtime performance
Easy maintenance | Eliminates repetitive boilerplate code
Seamless integration | Works perfectly with Lombok and Spring Boot
Clean & professional | Produces readable, production-ready code

## Conclusion
MapStruct bridges the gap between clarity and performance, giving
developers the speed of automation with the control of handwritten
code.
This document is a quick, basic overview of MapStruct — the library
offers many more advanced features and customization options beyond
what’s shown here.
From my perspective, it’s an incredibly powerful and reliable tool that I
use in my everyday development work to keep codebases clean, typesafe, and maintainable.


- [Home](./../../README.md)
- [Spring Tutorials](./../tutorials.md)
