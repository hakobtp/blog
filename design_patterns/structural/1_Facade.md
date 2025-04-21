# Facade

```info
Author      Ter-Petrosyan Hakob
```

---

The **Facade** pattern is a structural design pattern that provides a simplified interface to a set of complex subsystems. 
It makes the system easier to use by hiding internal complexity behind a single entry point. While similar to the 
Adapter and Decorator patterns, the primary goal of a facade is to offer one clear interface to multiple features.

## Motivation

As software grows in complexity, clients may need to interact with many classes or subsystems in a specific sequence. 
Without a facade, clients must know the subsystem APIs and the order of calls, leading to tight coupling and fragile code. 
A facade decouples clients from subsystem classes and reduces the number of dependencies.


## Structure

 ```mermaid
classDiagram
    class Facade {
        - CPU cpu
        - Memory memory
        - HardDrive hardDrive
        + start()
    }
    class CPU {
        + freeze()
        + jump(position)
        + execute()
    }
    class Memory {
        + load(position, data)
    }
    class HardDrive {
        + read(lba, size)
    }
    Facade --> CPU
    Facade --> Memory
    Facade --> HardDrive

```

## Example: Starting a Computer

Here is a Java implementation demonstrating the facade:

```java
// Subsystem 1
class CPU {
    public void freeze()         { /* ... */ }
    public void jump(long pos)   { /* ... */ }
    public void execute()        { /* ... */ }
}

// Subsystem 2
class Memory {
    public void load(long pos, byte[] data) { /* ... */ }
}

// Subsystem 3
class HardDrive {
    public byte[] read(long lba, int size)  { /* ... */ return new byte[size]; }
}

// Facade
public class ComputerFacade {
    private final CPU cpu = new CPU();
    private final Memory memory = new Memory();
    private final HardDrive hardDrive = new HardDrive();
    private static final long BOOT_ADDRESS = 0x00;
    private static final long BOOT_SECTOR  = 0x10;
    private static final int SECTOR_SIZE   = 512;

    public void start() {
        cpu.freeze();
        byte[] bootData = hardDrive.read(BOOT_SECTOR, SECTOR_SIZE);
        memory.load(BOOT_ADDRESS, bootData);
        cpu.jump(BOOT_ADDRESS);
        cpu.execute();
    }
}

// Client
public class Main {
    public static void main(String[] args) {
        new ComputerFacade().start();
    }
}
```

## Sequence Diagram

```mermaid
sequenceDiagram
    participant Client
    participant Facade
    participant CPU
    participant HD as HardDrive
    participant Mem as Memory

    Client ->> Facade: start()
    Facade ->> CPU: freeze()
    Facade ->> HD: read(BOOT_SECTOR, SECTOR_SIZE)
    Facade ->> Mem: load(BOOT_ADDRESS, bootData)
    Facade ->> CPU: jump(BOOT_ADDRESS)
    Facade ->> CPU: execute()
```

## When to Use

- **Complex subsystems:** When clients need many calls to work with a subsystem.
- **Reduce dependencies:** To decouple clients from subsystem classes.
- **Improve readability:** To present a clear API for common tasks.

---

## 📌 Explore More

- 🏠 [Home](./../../README.md)
- 🎨 [ Design Patterns](./../tutorials.md)