# Proxy
```info
Author      Ter-Petrosyan Hakob
```

---

The Proxy design pattern is a structural pattern. It is one of the simplest patterns to understand and use.

The Gang of Four defines the proxy pattern as:

> Provide a surrogate or placeholder for another object to control access to it.

In other words, a proxy acts as a gatekeeper. It sits between a client and a real object. The proxy can:

- Check permissions before operations
- Log or monitor requests
- Limit or modify access
- Cache results or delay object creation

This helps you protect sensitive actions or manage resource use without changing the real object or the client code.

**Class Diagram:**
<p align="center">
    <img src="./assets/img6.png" alt="img6" width="400"/>
</p>


This class diagram illustrates the structure of the Proxy pattern:

- `CommandExecutor` is an interface defining `runCommand(cmd)`.
- `CommandExecutorImpl` is the real object that implements `CommandExecutor`.
- `CommandExecutorProxy` also implements `CommandExecutor` and controls access before delegating to the real implementation.
- The Client uses the proxy rather than the real object directly, ensuring safe access.

## Example: Command Executor

Imagine a class that runs system commands. If you give it directly to a client, they could run dangerous commands like `rm -rf /`. 
We can use a proxy to prevent that.

**CommandExecutor Interface:**
```java
public interface CommandExecutor {
    void runCommand(String cmd) throws Exception;
}
```

**Real Command Executor:**
```java
public class CommandExecutorImpl implements CommandExecutor {
    @Override
    public void runCommand(String cmd) throws UnauthorizedException, CommandExecutionException {
        // Execute the system command
        Runtime.getRuntime().exec(cmd);
        System.out.println("Executed: '" + cmd + "'");
    }
}
```

**Proxy Class:**
```java
public class CommandExecutorProxy implements CommandExecutor {
    private static final String ADMIN_USER = "Gurgen";
    private static final String ADMIN_PWD = "J@urnalD$v";

    private final CommandExecutor executor;
    private final boolean isAdmin;

     public CommandExecutorProxy(String user, String pwd) {
        Objects.requireNonNull(user, "user cannot be null");
        Objects.requireNonNull(pwd,  "pwd cannot be null");

        this.executor = new CommandExecutorImpl();
        this.isAdmin  = ADMIN_USER.equals(user) && ADMIN_PWD.equals(pwd);
    }

    @Override
    public void runCommand(String cmd) throws UnauthorizedException, CommandExecutionException {
        Objects.requireNonNull(cmd, "cmd cannot be null");
        String trimmed = cmd.trim();

        if (!isAdmin && trimmed.startsWith("rm")) {
            throw new UnauthorizedException("Permission denied: non‑admin users cannot execute 'rm' commands.");
        }

        try {
            executor.runCommand(cmd);
        } catch (Exception e) {
            throw new CommandExecutionException("Error executing command: " + trimmed, e);
        }
    }
}
```

**Client Code:**
```java
public class ProxyPatternTest {
    public static void main(String[] args) {
        CommandExecutor executor = new CommandExecutorProxy("Gurgen", "wrong_pwd");
        try {
            executor.runCommand("ls -ltr");
            executor.runCommand("rm -rf abc.pdf");
        } catch (Exception e) {
            System.out.println("Error: " + e.getMessage());
        }
    }
}
```

In this example:
- The client uses `CommandExecutorProxy` instead of `CommandExecutorImpl`.
- The proxy allows safe commands for all users.
- It blocks dangerous commands for non-admin users.

**Sequence Diagram**

<p align="center">
    <img src="./assets/img7.png" alt="img7" width="700"/>
</p>

This sequence diagram shows how the Proxy handles a client request:

1) The Client calls `runCommand(cmd)` on the `CommandExecutorProxy`.
2) The Proxy checks if the user is an admin:
    - If `admin`, it delegates the call to `CommandExecutorImpl` and returns the result.
    - If not `admin`, it throws an exception for disallowed commands.


## Common Uses of Proxy Pattern

---

## 📌 Explore More

- 🏠 [Home](./../../README.md)
- 🎨 [ Design Patterns](./../tutorials.md)