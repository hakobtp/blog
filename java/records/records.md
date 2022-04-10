# Java Record

```info
Author :        Ter-Petrosyan Hakob
Last Updated:   2022-04-10
````

- [Immutable class](#immutable-class)
- [Record example](#record-example)

---

## Immutable class

Let's write an immutable class, for that we should  maintain the next rules

- private, final field for each piece of data
- getter for each field
- public constructor with a corresponding argument for each field
- equals method that returns true for objects of the same class when all fields match
- hashCode method that returns the same value when all fields match
- toString method that includes the name of the class and the name of each field and its corresponding value

```java
package com.htp.blog;

import java.util.Objects;

public class Employee {

    private final String firstName;
    private final String lastName;
    private final String email;
    private final double salary;

    public Employee(String firstName, String lastName, String email, double salary) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.email = email;
        this.salary = salary;
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
        Employee employe = (Employee) o;
        return Double.compare(employe.salary, salary) == 0
                && Objects.equals(firstName, employe.firstName)
                && Objects.equals(lastName, employe.lastName)
                && Objects.equals(email, employe.email);
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

## Record example



[Home](./../../README.md) 
| [<< Java Tutorials](./../tutorials.md)



