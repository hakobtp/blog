# 📘 Java Record

```
Author: Ter-Petrosyan Hakob
```

- [Immutable Classes](#immutable-classes)
- [Record Example](#record-example)

---

## Immutable Classes

When writing an immutable class, it is important to follow these guidelines:

- Use private, final fields for each piece of data.
- Provide a getter for each field.
- Supply a public constructor with a corresponding parameter for each field.
- Override the equals method to return true when all corresponding fields in two objects match.
- Override the hashCode method so that identical objects produce the same hash code.
- Override the toString method to include the class name, the field names, and their corresponding values.

Consider the following example of an immutable Employee class:

```java
import java.util.Objects;

public final class Employee {

    private final String firstName;
    private final String lastName;
    private final String email;
    private final double salary;

    public Employee(String firstName, String lastName, String email, double salary) {
        this.firstName = firstName;
        this.lastName  = lastName;
        this.email     = email;
        this.salary    = salary;
    }

    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public String getEmail() {
        return email;
    }

    public double getSalary() {
        return salary;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Employee employee = (Employee) o;
        return Double.compare(employee.salary, salary) == 0 &&
               Objects.equals(firstName, employee.firstName) &&
               Objects.equals(lastName, employee.lastName) &&
               Objects.equals(email, employee.email);
    }

    @Override
    public int hashCode() {
        return Objects.hash(firstName, lastName, email, salary);
    }

    @Override
    public String toString() {
        return "Employee{" +
               "firstName='" + firstName + '\'' +
               ", lastName='" + lastName + '\'' +
               ", email='" + email + '\'' +
               ", salary=" + salary +
               '}';
    }
}
```

Although modern IDEs like IntelliJ IDEA help generate boilerplate code, the resulting class can appear cluttered. Starting with Java 17, you can use records to simplify the process.

## Record Example

Java records were introduced as a concise way to create data carrier classes—that is, classes whose main purpose is to contain and transfer data (often referred to as POJOs or DTOs).

Let's rewrite our Employee class as a record:

```java
public record Employee(String firstName, String lastName, String email, double salary) {
}
```

Using records, the following are automatically generated:
- Immutable fields
- A canonical constructor
- An accessor method for each component (without the get prefix)
- The **equals(Object o)** method
- The **hashCode()** method
- The **toString()** method

For example, consider this usage within a simple application:

```java
public class App {
    public static void main(String[] args) {

        var e1 = new Employee("Gamer", "Simson", "gamer@mail.com", 1400);
        var e2 = new Employee("Gamer", "Simson", "gamer@mail.com", 1400);
        var e3 = new Employee("Bart", "Simson", "bart@mail.com", 0);

        System.out.println(e1);
        System.out.println(e2);
        System.out.println(e3);

        System.out.println("e1 equals e2 = " + e1.equals(e2));
        System.out.println("e1 equals e3 = " + e1.equals(e3));

        System.out.println("e1 hash code = " + e1.hashCode());
        System.out.println("e2 hash code = " + e2.hashCode());
        System.out.println("e3 hash code = " + e3.hashCode());
    }
}
```

The expected log output might be:

```
Employee[firstName=Gamer, lastName=Simson, email=gamer@mail.com, salary=1400.0]
Employee[firstName=Gamer, lastName=Simson, email=gamer@mail.com, salary=1400.0]
Employee[firstName=Bart, lastName=Simson, email=bart@mail.com, salary=0.0]
e1 equals e2 = true
e1 equals e3 = false
e1 hash code = -1706248431
e2 hash code = -1706248431
e3 hash code = 685814257
```

> **NOTE:** Records provide accessor methods without the get prefix. For example, call `e1.firstName()` instead of `e1.getFirstName()`.

Records are subject to a few important rules:

- **Field Limitation:** You cannot add additional instance fields to a record beyond those declared in its header. 
    However, static methods, fields, and static initializers are permitted.
- **Immutability:** The fields of a record are **implicitly final**. Attempts to modify them (for example, via a setter method) result in a compilation error:

    ```java
    public record Employee(String firstName, String lastName, String email, double salary) {

        public void setFirstName(String firstName) {
            // Compilation error: cannot assign a value to final variable firstName
            this.firstName = firstName;
        }

        public void tryToChangeFirstName() {
            // Compilation error: cannot assign a value to final variable firstName
            this.firstName = firstName.toLowerCase();
        }
    }
    ```

- **Inheritance:** Records are implicitly final and cannot be subclassed. They also cannot extend any other class, but they can implement interfaces. For instance:

    ```java
    interface IEmployee {
        String firstName();
        String fullName();
    }

    public record Employee(String firstName, String lastName, String email, double salary) implements IEmployee {

        @Override
        public String fullName() {
            return firstName + " " + lastName;
        }
    }
    ```
    In this example, the compiler enforces the implementation of `fullName()`, whereas the automatically generated `firstName()` accessor already satisfies the IEmployee contract for that component.

- **Nested and Local Declarations:** Records can be nested within a class or defined locally within a method. In both cases, they are implicitly static.

    > **Note:** While static fields, initializers, and methods are allowed in records, instance variables and instance initializers are not.
    ```java
    public record User(UUID uuid) {
        // Compilation error: Instance fields in records must be static.
        List<String> roles = new ArrayList<>();
    }
    ```

- **Customizing Components:** You can explicitly override the automatically generated accessor methods, but they must have the same signature and behavior as the default ones.
    ```java
    public record Employee(String firstName, String lastName, String email, BigDecimal salary) {

        @Override
        public String firstName(){
            return firstName;
        }
    }
    ```
- **Compact Constructor:** Records allow you to write a compact constructor that omits the parameter list. This constructor is typically used to validate and preprocess arguments:        
    ```java
    public record Employee(String firstName, String lastName, String email, BigDecimal salary) {

        public Employee {
            Objects.requireNonNull(firstName);
            Objects.requireNonNull(lastName);
            Objects.requireNonNull(email);
            Objects.requireNonNull(salary);
        }
    }
    ```
    Remember, records provide only shallow immutability. If a component’s type is mutable (for example, a List or Date), the record itself will not prevent modifications. The compact constructor is a good place to enforce immutability by creating defensive copies:
    ```java
    public record User(UUID uuid, List<String> roles) {

        public User {
            roles = List.copyOf(roles);
        }
    }
    ```
- **Constructor Behavior:** The body of the compact constructor is executed before the final assignment of the field values. Consider this example:  
    ```java
    public record Employee(String firstName, String lastName, String email, BigDecimal salary) {

        public Employee {
            System.out.println("firstName: " + firstName()
                           + ", lastName: " + lastName()
                           + ", email: " + email()
                           + ", salary: " + salary());
        }
    }
    ```
    This might print null for all fields because the accessor methods have not yet been initialized with the final values. Although it may seem as if the final values are being modified, the compiler creates an intermediate placeholder and performs a single assignment at the end of the constructor.

- **Modifying Initialization Values:** You can also adjust the initialization values for components within the compact constructor:
    ```java
    public record Employee(String firstName, String lastName, String email, BigDecimal salary) {

        public Employee {
            // Modify email to always be in lowercase
            email = email.toLowerCase();
        }
    }
    ```
- **Canonical Constructor Replacement:** You can provide your own canonical constructor; however, remember that it must assign all declared components. For example:
    ```java
    public record Employee(String firstName, String lastName, String email, BigDecimal salary) {

        public Employee(String firstName, String lastName, String email, BigDecimal salary) {
            this.firstName = firstName;
            this.lastName  = lastName;
            this.email     = email;
            this.salary    = salary;
        }
    }
    ```
    Attempting to define a constructor that doesn’t initialize all fields leads to a compilation error:
    ```java
    public record Employee(String firstName, String lastName, String email, BigDecimal salary) {

        public Employee(String firstName, String lastName) {
            // Compilation error: the canonical constructor must initialize all components.
            this.firstName = firstName;
            this.lastName  = lastName;
        }
    }
    ```
- **Access Modifiers:** An explicit canonical constructor cannot have a more restrictive access level than the record itself:    
    ```java
    // Package-private record
    record Employee(String firstName, String lastName, String email, BigDecimal salary) {

        private Employee(String firstName, String lastName, String email, BigDecimal salary) {
            // Compilation error: Cannot assign stronger access privileges.
            this.firstName = firstName;
            this.lastName  = lastName;
            this.email     = email;
            this.salary    = salary;
        }
    }
    ```  
- **Copying a Record:** To create a copy of a record, you must explicitly pass all the fields to the constructor: 
    ```java
    public class App {

        public static void main(String[] args) {
            var gamer = new Employee("Gamer", "Simson", "gamer@mail.com", BigDecimal.valueOf(23));
            var gamerCopy = new Employee(gamer.firstName(), gamer.lastName(), gamer.email(), gamer.salary());
        }
    }
    ```         

---

- 🏠 [Home](./../../README.md)
- ☕ [Java Tutorials](./../tutorials.md)