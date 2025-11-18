# Using Annotations at Runtime

```
Author: Ter-Petrosyan Hakob
```

---

So far, you have learned how to add annotations to your Java code and how to create your own annotation types. Now, let’s see how we can actually use those annotations while the program is running. This is called runtime annotation processing, and it can make your code more flexible and easier to maintain.

## Why Process Annotations at Runtime?

Imagine you often need to create `toString` methods for your classes. These methods return a text representation of an object, 
usually showing the values of its fields. Writing them by hand can be repetitive. You could write a generic `toString` using reflection that 
lists every field automatically—but sometimes you want more control.

For example, consider a Coordinate class:

- A generic `toString` might return `Coordinate[x=5, y=10]`.
- Maybe you prefer a simpler format like `[5, 10]`.
- Or you might want to include only some fields, not all of them.

Annotations allow you to mark which classes and fields should be included in the custom string format.

### Defining a @ToString Annotation

First, we define the `@ToString` annotation like this:

```java
import java.lang.annotation.*;

@Target({ElementType.FIELD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
public @interface ToString {
    boolean includeName() default true;  
}
```

- `@Target` specifies where the annotation can be used (on classes and fields here).
- `@Retention(RetentionPolicy.RUNTIME)` means the annotation will be available at runtime.
- `includeName` is a property that decides whether the field or class name appears in the string.

### Using the Annotation on Classes

Here’s an example with two classes, `Coordinate` and `Box`:

```java
@ToString(includeName=false)
public class Coordinate {
    @ToString(includeName=false) private int x;
    @ToString(includeName=false) private int y;

    public Coordinate(int x, int y) {
        this.x = x;
        this.y = y;
    }
}

@ToString
public class Box {
    @ToString(includeName=false) Coordinate topLeft;
    @ToString int width;
    @ToString int height;

    public Box(Coordinate topLeft, int width, int height) {
        this.topLeft = topLeft;
        this.width = width;
        this.height = height;
    }
}
```

If we annotate a class with `@ToString`, it can be processed by a runtime method that automatically creates a string. 

For a `Box`, the string might look like:
```
Box[[5, 10], width=20, height=30]
```

### Processing Annotations at Runtime

We cannot change the `toString` method of a class at runtime. But we can write a utility method that reads the annotations and formats objects accordingly.

Java’s reflection API provides methods to access annotations, such as:

- `getAnnotation(Class<T>)` – Get a specific annotation.
- `getAnnotationsByType(Class<T>)` – Get repeated annotations.
- `isAnnotationPresent(Class<? extends Annotation>)` – Check if an annotation exists.
- `getDeclaredFields()` – Get all fields in a class.

Here’s the main idea:
```java
import java.lang.reflect.*;

public class ToStringUtil {
    public static String toString(Object obj) {
        if (obj == null) {
            return "null";
        }

        Class<?> cls = obj.getClass();
        ToString ts = cls.getAnnotation(ToString.class);
        if (ts == null) {
            return obj.toString();
        }

        StringBuilder sb = new StringBuilder();
        if (ts.includeName()) {
            sb.append(cls.getSimpleName());
        }
        sb.append("[");

        boolean first = true;
        for (Field f : cls.getDeclaredFields()) {
            ts = f.getAnnotation(ToString.class);
            if (ts != null) {
                if (first) {
                    first = false;
                } else {
                    sb.append(",");
                }
                f.setAccessible(true);
                if (ts.includeName()) {
                    sb.append(f.getName()).append("=");
                }
                try {
                    sb.append(toString(f.get(obj)));  // recursive call
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                }
            }
        }
        sb.append("]");
        return sb.toString();
    }
}
```

How It Works

1. The method checks if the object’s class has a `@ToString` annotation.
2. If it does, it loops over the fields and looks for `@ToString` annotations on them.
3. If a field is annotated, it includes the field in the output string.
4. The method calls itself recursively if a field is an object that also has annotations.
5. If a class or field is not annotated, the normal toString is used.


### Using the Utility

```java
public class RuntimeAnnotationDemo {
    public static void main(String[] args) {
        var box = new Box(new Coordinate(5, 10), 20, 30);
        System.out.println(ToStringUtil.toString(box));
    }
}
```

Output:

```
Box[[5,10],width=20,height=30]
```

### Key Takeaways

- Runtime annotations let you customize behavior without changing the class code.
- Use reflection methods like `getAnnotation()` and `getDeclaredFields()` to read annotations.
- Recursive formatting allows you to handle nested objects.
- This approach is flexible: you can create custom rules, skip certain fields, or format them differently.

Annotations are powerful tools for simplifying repetitive tasks and adding metadata to your code. They work best when combined with reflection for runtime processing.


--- 

- `java.lang.reflect.AnnotatedElement`
    - `boolean isAnnotationPresent(Class<? extends Annotation> annotationType)` returns `true` if this element has an annotation of the specified type.
    - `<T extends Annotation> T getAnnotation(Class<T> annotationType)` returns the annotation of the given type, or `null` if this element does not have it.
    - `<T extends Annotation> T[] getAnnotationsByType(Class<T> annotationType)`returns all annotations of a repeatable type. If there are none, it returns an empty array (length 0).
    - `Annotation[] getAnnotations() returns` all annotations that exist on this element, including inherited ones. If there are none, it returns an empty array.    
    - `Annotation[] getDeclaredAnnotations()` returns all annotations that exist on this element, including inherited ones. If there are none, it returns an empty array.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Annotations in the Java API](./3_Annotations_in_the_Java_API.md)
- [Source-Level Annotation Processing](./5_Source_Level_Annotation_Processing.md)