# Build lifecycle phases

Maven's default build lifecycle consists of several phases, each responsible for a specific step in building, 
testing, and deploying your project. Here’s a detailed explanation of the key phases:

- **validate**  
    - **Purpose:** The validate phase checks if the project structure is correct and verifies that all necessary 
        information (like the `pom.xml` configuration) is available. It’s the very first step in the build process.
    - **What Happens:** Ensures that the project’s directory structure meets Maven’s standards.
        Checks that all required configurations and dependencies are specified.
    - **Example:** If a required property is missing in your `pom.xml`, validate will catch it before any further processing.
- **compile**
    - **Purpose:** The compile phase compiles the project’s source code into bytecode (typically producing `.class` files).
    - **What Happens:** Maven invokes the **maven-compiler-plugin** (or another compiler plugin) to transform your Java source 
    `code into compiled code. This phase only deals with main source code, not test code.
    - **Example:** Your Java files in `src/main/java` are compiled into the `target/classes` directory.
- **test**
    - **Purpose:** The test phase runs the unit tests to ensure the compiled code works as expected.
    - **What Happens:** Maven uses the **maven-surefire-plugin** to execute tests.
        It runs tests located in **src/test/java** (or another specified location) against the compiled code.
        This phase is crucial for catching errors early, ensuring that changes haven't broken the functionality.
    - **Example:** JUnit tests are executed, and if any tests fail, the build process stops (unless configured to continue).
- **package**
    - **Purpose:** The package phase takes the compiled code and packages it into a distributable format.
    - **What Happens:** The project is packaged into a JAR, WAR, or another archive format.
        Plugins like the **maven-jar-plugin** or **maven-war-plugin** are used to create the final artifact.
    - **Example:** After compiling and testing, your application is packaged as a `my-app.jar` or `my-webapp.war` file in the target directory.
- **install**
    - **Purpose:** The install phase installs the packaged artifact into your local Maven repository.
    - **What Happens:** The artifact (e.g., JAR or WAR) is copied to your local repository (typically located in your 
        user home directory under .**m2/repository**). This allows you to use your built artifact as a dependency in other local projects.
    - **Example:** If you build a library project, after running **mvn install**, other projects on your machine can reference that 
        library by its group ID, artifact ID, and version.
- **deploy**
    - **Purpose:** The deploy phase is used in a distributed development environment to copy the final package to a remote repository.
    - **What Happens:** The artifact is uploaded to a remote repository manager (like Nexus, Artifactory, or another Maven repository server).
        This makes the artifact available for other developers and projects within your organization.
    - **Example:** When you run mvn deploy in a CI/CD pipeline, your application is automatically published to your company’s 
        remote repository for shared use.

### Summary

- **validate:** Ensures the project setup is correct.
- **compile:** Compiles your source code.
- **test:** Executes unit tests to verify code correctness.
- **package:** Bundles the compiled code into a distributable format.
- **install:** Installs the package into the local repository.
- **deploy:** Publishes the package to a remote repository for broader use.

Each phase is designed to work sequentially, ensuring a smooth and automated transition from source code to a deployed artifact, with testing and packaging integrated into the process.

---

- [Home](./../../README.md)
- [Maven tutorials](./../tutorials.md)