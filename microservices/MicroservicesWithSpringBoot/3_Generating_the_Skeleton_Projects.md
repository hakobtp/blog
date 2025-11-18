# Generating the Skeleton Projects

Before we start writing our microservices, we first need a skeleton project. A skeleton project is like a starting template: 
it contains the necessary files for building the project and includes an empty main class and test class for the microservice.

Later, we will see how to build all our microservices at once using a multi-project setup with Gradle.

## Creating Each Microservice

For our Education platform, we will create four Spring Boot microservices:

- **Course Service** – manages courses
- **Chapter Service** – manages chapters of courses
- **Quiz Service** – manages quizzes and exercises
- **Course Composite Service** – combines information from the other three services

Each microservice will:

- Use **Gradle** as the build tool
- Use **Java 17**
- Be packaged as a fat **JAR** file (includes all dependencies)
- Include dependencies for **Spring Boot Actuator** and **Spring WebFlux**
- Use Spring Boot v3.5.7

Spring Boot Actuator provides helpful endpoints to monitor and manage your microservices.
Spring WebFlux allows us to create REST APIs to communicate between services.

**Step 1: Create a Directory**

First, create a main directory for all microservices:

```bash
mkdir microservices-with-spring-boot && cd microservices-with-spring-boot
mkdir microservices && cd microservices
```

**Step 2: Generate Projects with Spring Initializr**

Now, we can generate each microservice using the Spring Initializr CLI:

```bash
spring init \
--boot-version=3.5.7 \
--type=gradle-project \
--java-version=17 \
--packaging=jar \
--name=course-service \
--package-name=com.htp.microservices.core.course \
--groupId=com.htp.microservices.core.course \
--dependencies=actuator,webflux \
--version=1.0.0-SNAPSHOT \
course-service

spring init \
--boot-version=3.5.7 \
--type=gradle-project \
--java-version=17 \
--packaging=jar \
--name=chapter-service \
--package-name=com.htp.microservices.core.chapter \
--groupId=com.htp.microservices.core.chapter \
--dependencies=actuator,webflux \
--version=1.0.0-SNAPSHOT \
chapter-service

spring init \
--boot-version=3.5.7 \
--type=gradle-project \
--java-version=17 \
--packaging=jar \
--name=quiz-service \
--package-name=com.htp.microservices.core.quiz \
--groupId=com.htp.microservices.core.quiz \
--dependencies=actuator,webflux \
--version=1.0.0-SNAPSHOT \
quiz-service

spring init \
--boot-version=3.5.7 \
--type=gradle-project \
--java-version=17 \
--packaging=jar \
--name=course-composite-service \
--package-name=com.htp.microservices.core.course_composite \
--groupId=com.htp.microservices.core.course_composite \
--dependencies=actuator,webflux \
--version=1.0.0-SNAPSHOT \
course-composite-service
```

**Step 3: File Structure**

After running these commands, your folder structure will look like this:

```
microservices-with-spring-boot
    |--microservices
        |-- course-service
        |-- chapter-service
        |-- quiz-service
        |-- course-composite-service
```

Each folder contains a basic Spring Boot project ready to build and run.

**Step 4: Build the Microservices**

You can build each microservice separately using Gradle Wrapper (gradlew):

```bash
$ pwd

/microservices-with-spring-boot/microservices


cd chapter-service; ./gradlew build; cd -;
cd course-service; ./gradlew build; cd -;
cd course-composite-service; ./gradlew build; cd -;
cd quiz-service; ./gradlew build; cd -;
```

> **Notes:**
> - `gradlew` is created by Spring Initializr, so you **don’t need Gradle installed**.
> - The first time you run `gradlew`, it will download Gradle automatically.
> - The version of Gradle used is defined in `gradle/wrapper/gradle-wrapper.properties` in each project.

## Setting Up Multi-Project Builds with Gradle

Sometimes, it is useful to build all microservices at once instead of building each one separately. Gradle allows us to do this using a multi-project build.

A multi-project build treats multiple projects as a single system, so we can run one command to build everything.

**Step 1: Create settings.gradle**

First, we need a settings file to tell Gradle which projects belong to the build. Create a file called `settings.gradle` 
in the root folder (`microservices-with-spring-boot`) and add the following:

```groovy
rootProject.name = 'microservices-with-spring-boot'

include ':microservices:chapter-service'
include ':microservices:course-service'
include ':microservices:course-composite-service'
include ':microservices:quiz-service'
```

This tells Gradle which microservices it should include when building.

**Step 2: Reuse Gradle Wrapper**

Next, we need the Gradle Wrapper files in the root folder. These files allow us to run Gradle commands without installing Gradle manually.

Copy the Gradle wrapper and related files from one of the microservices:

```bash
pwd
/microservices-with-spring-boot

cp -r microservices/chapter-service/gradle .
cp microservices/chapter-service/gradlew .
cp microservices/chapter-service/gradlew.bat .
cp microservices/chapter-service/.gitignore .

```

**Step 3: Clean Up Individual Projects**

Now that the wrapper exists in the root folder, we no longer need the Gradle wrapper files in each microservice. Remove them with:

```bash
pwd 
/microservices

find . -mindepth 2 -maxdepth 2 -type d -name "gradle" -exec rm -rv "{}" +
find ./*/ -maxdepth 1 -type f -name "gradlew*" -exec rm -v "{}" \;
find ./*/ -maxdepth 1 -type f -name "HELP*" -exec rm -v "{}" \;
```

**Step 4: Build All Microservices at Once**

Finally, we can build all projects with one command from the root folder:

```bash
pwd
/microservices-with-spring-boot

./gradlew build
```

Gradle will go through each microservice and build it automatically.

```
microservices-with-spring-boot
    |-- gradle
    |-- gradlew
    |-- gradlew.bat
    |-- settings.gradle
    |--.gitignore
    |--microservices
        |-- course-service
        |-- chapter-service
        |-- quiz-service
        |-- course-composite-service
```

## Note on DevOps Best Practices

In real-world projects, each microservice often has its own build and release cycle. This allows teams to deploy and update microservices independently.

However, for learning and testing purposes, a multi-project setup is simpler because it lets us build and deploy the entire system with a single command.


The source code for this article is available [over on GitHub](https://github.com/hakobtp/microservices-with-spring-boot/tree/skeleton_projects){:target="_blank" rel="noopener"}.

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Exploring Our Education Microservices](./2_Exploring_Our_Education_Microservices.md)
- [Api and Util libraries](./4_Api_and_Util_libraries.md)