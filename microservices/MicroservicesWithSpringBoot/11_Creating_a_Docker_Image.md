# Creating a Docker Image

Before we create a Docker image, it’s important to understand how Java behaves when it runs inside a container. 
In this section, we will see how to limit `CPU` cores and `memory` for a Java process. This helps us create predictable and efficient applications.

We will start with CPU limits and then look at memory limits.

## Limiting Available CPUs

Java can detect how many CPU cores are available to it. By default, it sees all the CPU cores of the machine unless we limit them through Docker.

First, let’s check how many processors Java sees when no limits are applied.
We can do this by sending the line:

```java
Runtime.getRuntime().availableProcessors()
```

to `jshell`, the Java command-line tool.

We run `jshell` inside a Docker container that uses the `eclipse-temurin:17` image:

```bash
echo 'Runtime.getRuntime().availableProcessors()' \
| docker run --rm -i eclipse-temurin:17 jshell -q
```

A typical output might look like this

```bash
jshell> $1 ==> 8
```

This means Java detected 8 CPU cores, which matches the host machine.

Now let’s limit the container to 3 CPUs using the `--cpus` option:

```bash
echo 'Runtime.getRuntime().availableProcessors()' \
| docker run --rm -i --cpus=3 eclipse-temurin:17 jshell -q
```

Output:
```bash
jshell> $1 ==> 3
```
Java now recognizes only 3 processors. This shows that Java respects Docker’s CPU limits.
This is useful because Java uses the number of processors to size internal components, like thread pools.

## Limiting Available Memory

Next, let’s look at memory. We want to see how much heap space Java thinks it can use.

We can ask Java for its internal settings by running:

```bash
docker run -it --rm eclipse-temurin:17 \
  java -XX:+PrintFlagsFinal | grep "size_t MaxHeapSize"
```

Example output:
```bash
size_t MaxHeapSize = 4160749568
```

This number is in bytes. If we convert it: `4,160,749,568 bytes ≈ 3,968 MB ≈ 4 GB`

---

- [Home](./../../README.md)
- [Microservices](./../tutorials.md)
- [Adding Isolated Tests to the Microservices](./10_Adding_Isolated_Tests_to_the_Microservices.md)