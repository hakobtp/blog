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



- [Home](./../../README.md)
- [Spring Tutorials](./../tutorials.md)
