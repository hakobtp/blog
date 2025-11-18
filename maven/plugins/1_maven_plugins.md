## Maven plugins 

Maven plugins are like small tools that extend Maven’s capabilities. They are pieces of code (packaged as JAR files) 
that perform specific tasks during the build process, such as compiling code, running tests, packaging artifacts, generating documentation, and more.

### How Maven Plugins Work

1) Integration into the Build Lifecycle:
    - Maven has a well-defined build lifecycle with phases like **validate**, **compile**, **test**, **package**, **install**, and **deploy**.
    - Plugins are bound to these phases. For example, the **maven-compiler-plugin** is bound to the **compile** phase, and the 
        **maven-surefire-plugin** is bound to the test phase.
    - When you run a Maven command (e.g., **mvn package**), Maven goes through the phases in order and executes the goals (tasks) 
        provided by the plugins configured for those phases.
2) Goals:
    - Each plugin can have one or more goals. A goal is a specific task the plugin can perform.
    - For example, the **maven-jar-plugin** has a goal that packages compiled code into a JAR file.
    - You can also run goals directly from the command line, like **mvn compiler:compile**, which explicitly 
        runs the compile goal of the **maven-compiler-plugin**.        
3) Configuration:
    - Plugins are configured in your project's `pom.xml` file under the **<build>** section.
    - Here, you can specify which plugins to use, set their versions, and provide any necessary configuration 
        details (e.g., source and target Java versions for the compiler plugin).        

### Which Parts of a Maven Project Use Plugins

- **Build Process:** Most plugins are involved in the build process. They handle tasks like:
    - Compiling: **maven-compiler-plugin**
    - Testing: **maven-surefire-plugin** for unit tests and **maven-failsafe-plugin** for integration tests
    - Packaging: **maven-jar-plugin**, **maven-war-plugin**, etc.
    - Deploying: **maven-deploy-plugin**
- **Reporting and Documentation:** Some plugins generate reports or documentation, such as:
    - Site Generation: **maven-site-plugin**
    - Javadoc Generation: maven-javadoc-plugin
- **Other Tasks:** There are plugins for code quality, dependency analysis, versioning, and more.

### Example of a Plugin Configuration in pom.xml

Here’s a simple example using the **maven-compiler-plugin**:

```xml
<build>
  <plugins>
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-compiler-plugin</artifactId>
      <version>3.8.1</version>
      <configuration>
        <source>11</source>
        <target>11</target>
      </configuration>
    </plugin>
  </plugins>
</build>
```
- groupId, artifactId, and version: Identify the plugin.
- configuration: Sets parameters, such as the Java version.

### Summary

- **What Is a Maven Plugin?** It’s an add-on that performs specific tasks during the Maven build process.
- **How Do They Work?** They’re bound to the Maven lifecycle phases, executing their goals at the right time in the build sequence.
- **Where Are They Used?** Primarily in the **<build>** section of your `pom.xml`, but there are also plugins for reporting and documentation.

Understanding Maven plugins is key to mastering Maven, as they are the workhorses that automate most of your build and deployment tasks.

## Goals

Maven plugins are made up of one or more goals—individual tasks that the plugin can perform. 
Understanding how goals work is essential to mastering Maven. Here’s a deeper dive into the role and configuration of goals:

## What Is a Goal?

- **Definition:** A goal is a specific, executable unit of work within a Maven plugin. 
    Think of it as a command that performs a particular task. For example, the **compile** goal in the **maven-compiler-plugin** compiles your source code.
- **Granularity:** Goals are designed to be small and focused. Each goal usually handles a single task 
    (like packaging code, generating documentation, or running tests).
- **Multiple Goals per Plugin:** A single Maven plugin can provide several goals. For instance, the 
    **maven-compiler-plugin** offers goals such as **compile** (for compiling main sources) and **testCompile** (for compiling test sources).

### Binding Goals to Build Phases

- **Maven Build Lifecycle:** Maven defines a series of phases (like **validate**, **compile**, **test**, **package**, **install**, and **deploy**). 
    Plugins and their goals are bound to these phases, which means that when you run a phase, Maven automatically executes the associated goals.
- **Default Bindings:** Many plugins come with default bindings. For example, the **compile** goal of the 
    **maven-compiler-plugin** is automatically executed during the **compile** phase, and the test goal of the 
    **maven-surefire-plugin** runs during the **test** phase.
- **Custom Bindings:** You can also customize which phase a goal is bound to. This is done in the **<executions>** section of 
    the plugin configuration in your `pom.xml`. 
    
For instance:

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-failsafe-plugin</artifactId>
  <version>2.22.2</version>
  <executions>
    <execution>
      <id>integration-tests</id>
      <goals>
        <goal>integration-test</goal>
        <goal>verify</goal>
      </goals>
      <phase>integration-test</phase> <!-- Explicitly binding to a phase -->
    </execution>
  </executions>
</plugin>
```

Here, the goals **integration-test** and **verify** are explicitly bound to the **integration-test** phase 
(and later the verify phase, as Maven moves sequentially).

### Running Goals Directly

**Direct Invocation:** You’re not limited to having goals execute only during the lifecycle phases. You can run a specific goal directly from the command line. For example:

```xml
mvn compiler:compile
```    
This command directly invokes the compile goal of the maven-compiler-plugin, regardless of the lifecycle phase.


**Selective Execution:** Running goals directly can be very useful when you need to perform a specific task without going through 
the entire build lifecycle. For example, you might run tests, compile code, or generate documentation on demand.

### Plugin Configuration and Goals

**Configuration Block:** In your `pom.xml`, you configure plugins (and their goals) under the **`<build>`** section. 
The configuration not only defines which goals to run but also sets parameters that control how these goals execute.

Here’s an example for the **maven-compiler-plugin**:

```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-compiler-plugin</artifactId>
  <version>3.8.1</version>
  <configuration>
    <source>11</source>
    <target>11</target>
  </configuration>
</plugin>
```

In this configuration, the plugin’s goals (by default, **compile** and **testCompile**) will use Java 11 as specified.

**Executions:** The **`<executions>`** element allows you to fine-tune which goals are bound to which phases and to customize 
their behavior further. This is useful when you need to run multiple instances of the same goal or change when it’s executed.

### Common Maven Plugin Goals

- **maven-compiler-plugin:**
    - **compile**: Compiles the main source code.
    - **testCompile**: Compiles the test source code.
- **maven-surefire-plugin:**
    - **test**: Runs unit tests during the test phase.
- **maven-failsafe-plugin:**
    **integration-test**: Executes integration tests during the **integration-test** phase.
    **verify**: Checks the results of integration tests during the verify phase.
- **maven-jar-plugin**:
    **jar**: Packages compiled code into a JAR file during the package phase.

### Summary

- Goals are the building blocks of Maven plugins, each handling a specific task.
- They are bound to phases of the Maven build lifecycle by default or via custom configuration.
- You can run goals directly from the command line for targeted operations.
- Configuration in your `pom.xml` lets you customize when and how these goals execute.

Understanding how goals work—and how they integrate into the Maven lifecycle—allows you to harness Maven's full power to automate your build, testing, and deployment processes.

---

- [Home](./../../README.md)
- [Maven tutorials](./../tutorials.md)