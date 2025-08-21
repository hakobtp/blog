# Setting Up Hibernate with Maven (Step-by-Step Guide)

```info
Author      Ter-Petrosyan Hakob
```

---

To use Hibernate in your project, you need to add some dependencies to your Maven file (`pom.xml`). 
Dependencies are like tools or libraries that your project needs to work properly.

Here is a simple `pom.xml` file with the basic dependencies for a Hibernate project:

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.hakobtp.blog</groupId>
    <artifactId>hibernate-example</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- Hibernate Core -->
        <dependency>
            <groupId>org.hibernate.orm</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>6.4.4.Final</version>
        </dependency>

        <!-- H2 Database for testing -->
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <version>2.2.224</version>
            <scope>runtime</scope>
        </dependency>

        <!-- JPA API -->
        <dependency>
            <groupId>jakarta.persistence</groupId>
            <artifactId>jakarta.persistence-api</artifactId>
            <version>3.1.0</version>
        </dependency>

        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.30</version>
            <scope>provided</scope>
        </dependency>

        <!-- SLF4J API -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>2.0.9</version>
        </dependency>

        <!-- SLF4J Simple Logger -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>2.0.9</version>
        </dependency>

        <!-- JUnit 5 -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-api</artifactId>
            <version>5.10.1</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter-engine</artifactId>
            <version>5.10.1</version>
            <scope>test</scope>
        </dependency>

        <!-- Mockito -->
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <version>5.11.0</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-junit-jupiter</artifactId>
            <version>5.11.0</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Compiler Plugin for Lombok -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>17</source>
                    <target>17</target>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>1.18.30</version>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>

            <!-- Surefire Plugin for running JUnit 5 tests -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.2.5</version>
            </plugin>
        </plugins>
    </build>
</project>
```

**What This `pom.xml` File Does:**

- Sets up Hibernate to work with JPA, so you can map Java classes to database tables.
- Uses H2, a lightweight in-memory database, perfect for testing and learning.
- Enables Lombok support during Maven builds, saving you from writing boilerplate code like getters, setters, and constructors.
- Adds SLF4J logging support, so you can easily use `@Slf4j` from Lombok to log messages in your classes.
- Includes JUnit 5 for writing unit tests with modern annotations like `@Test`, `@BeforeEach`, and `@DisplayName`.
- Includes Mockito, which helps you create mock objects for testing classes in isolation.
- Adds the Surefire Plugin, which allows Maven to run your tests automatically when you build the project.

## Creating and Testing a Simple JPA Entity with Hibernate

Let’s start with a very basic example. Don’t worry if this looks simple — it will make more sense as we go further in the series.

**Step 1: The Author Entity**

Suppose we have an author who must have:
- a first name
- a last name
- an email
- and an about me section

Here's how we can create an entity class for the author:

```java
@Entity
@Getter
@Setter
@ToString
@Accessors(chain = true)
@Table(name = "authors")
public class AuthorEntity {
    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false, length = 50)
    private String firstName;

    @Column(nullable = false, length = 50)
    private String lastName;

    private String email;
    private String aboutMe;
}
```

> **Hint:** When we save an AuthorEntity into the database, Hibernate will generate the ID automatically. 
> If the id is not null after saving, it means the author was successfully stored.

An entity is a Java class that JPA treats as a table in the database. To make a class an entity, follow these rules:

- Put `@javax.persistence.Entity` above your class so JPA knows it represents data in a table.
- Use `@javax.persistence.Id` on a field to mark the primary key.
- The class must have a `public` or `protected` constructor with **no parameters**. You can add more constructors if you like.
- The entity cannot be an inner class, enum, or interface—it must be a regular, top-level class.
- The class itself and any persistent fields or methods must not be `final`, so JPA can create and modify instances.
- If you need to send entity objects over the network (for example, in a remote API), have the class implement `java.io.Serializable`. 
    This step is not required by JPA but can help in distributed setups.
    
With these rules, JPA can map your class to a database table and manage its data.    

**Step 2: The persistence.xml File**

We also need a configuration file so that Hibernate knows how to connect to the database. Create the file in:

```
src/main/resources/META-INF/persistence.xml
```

Here’s the content:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence
                                 http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd"
             version="2.2">

    <persistence-unit name="bookstore" transaction-type="RESOURCE_LOCAL">
        <!-- Use Hibernate as the JPA provider -->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>

        <!-- Automatically include all annotated classes -->
        <exclude-unlisted-classes>false</exclude-unlisted-classes>

        <properties>
            <!-- JDBC connection settings -->
            <property name="jakarta.persistence.jdbc.url" value="jdbc:h2:mem:bookStoreDB"/>
            <property name="jakarta.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="jakarta.persistence.jdbc.user" value="sa"/>
            <property name="jakarta.persistence.jdbc.password" value=""/>

            <!-- Automatically drop and recreate schema at startup -->
            <property name="jakarta.persistence.schema-generation.database.action" value="drop-and-create"/>

            <!-- Hibernate-specific settings -->
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
        </properties>
    </persistence-unit>
</persistence>
```

What this file does:

- Sets up an in-memory H2 database (no need to install anything)↳
- Uses Hibernate to manage our data
- Tells Hibernate to show SQL logs in the console
- Automatically drops and recreates the schema each time the application runs

**Step 3: Writing Tests for the Entity**

Let’s write a test class to check if our AuthorEntity works as expected.

```java
@Slf4j
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class AuthorPersistenceTest {

    // This factory creates EntityManager instances. It connects to our persistence unit.
    private static EntityManagerFactory emf;
    
    // EntityManager is used to interact with the database
    private EntityManager em;
    
    // Used to control database transactions (begin, commit, rollback)
    private EntityTransaction tx;

    // Runs once before all tests to set up the factory
    @BeforeAll
    void setUpFactory() {
        emf = Persistence.createEntityManagerFactory("bookstore");
    }

    // Runs once after all tests to close the factory
    @AfterAll
    void closeFactory() {
        if (emf != null) emf.close();
    }

    // Runs before each test to create a new EntityManager and transaction
    @BeforeEach
    void init() {
        em = emf.createEntityManager();
        tx = em.getTransaction();
    }

    // Runs after each test to close the EntityManager
    @AfterEach
    void close() {
        if (em != null) em.close();
    }

    @Test
    @DisplayName("Should save author to database")
    void shouldSaveAuthorToDatabase() {
        // Create a new author with valid data
        var author = new AuthorEntity()
                .setFirstName("Charles")
                .setLastName("Dickens");

        // The ID should be null before saving
        assertNull(author.getId(), "ID should be null before persisting");

        // Start a transaction and save the author
        tx.begin();
        em.persist(author);
        tx.commit();

        // After saving, the ID should not be null (it is generated by Hibernate)
        assertNotNull(author.getId(), "ID should not be null after persisting");
    }

    @Test
    @DisplayName("Should fail to save author without last name and not insert into DB")
    void shouldFailToSaveAuthorWithoutLastName() {
        // Create an author with missing last name (which is required)
        var author = new AuthorEntity()
                .setFirstName("Charles")
                .setLastName(null); // Invalid: lastName is required

        tx.begin();

        // Try to save the author. It should throw a PersistenceException.
        assertThrows(PersistenceException.class, () -> {
            em.persist(author);
            tx.commit(); // This will fail because of null lastName
        });

        // If transaction is still active, roll it back
        if (tx.isActive()) {
            tx.rollback();
        }

        // Now check that the author was not saved in the database
        tx.begin();
        List<AuthorEntity> authors = em
                .createQuery("SELECT a FROM AuthorEntity a", AuthorEntity.class)
                .getResultList();
        tx.commit();

        // Database should be empty
        assertTrue(authors.isEmpty(), "Author should not be saved in the database");
    }
}

```

Note: The name `"bookstore"` in the test must match the persistence-unit name in `persistence.xml`.


## Creating a JPA Repository for Authors

Now that we have our AuthorEntity and Hibernate configuration ready, let’s write a simple repository 
class to interact with the database using pure JPA (no manual SQL or JDBC).

We’ll create an `AuthorRepository` class with two basic methods:

- `save()` – to persist a new author
- `findById()` – to search for an author by their ID

`AuthorRepository.java`

```java
@RequiredArgsConstructor
public class AuthorRepository {

    private final EntityManager em;

    // Save a new author and return the generated ID
    public Long save(AuthorEntity authorEntity) {
        em.getTransaction().begin();      // Start a transaction
        em.persist(authorEntity);         // Tell JPA to save the entity
        em.getTransaction().commit();     // Commit the transaction
        return authorEntity.getId();      // Return the generated ID
    }

    // Find an author by ID. Returns Optional.empty() if not found
    public Optional<AuthorEntity> findById(Long id) {
        AuthorEntity author = em.find(AuthorEntity.class, id); // Fetch by primary key
        return Optional.ofNullable(author);                    // Wrap in Optional
    }
}

```
This repository uses the **EntityManager** provided by JPA to save and retrieve data, without writing any SQL.

**Testing the Repository**

Let’s now write some unit tests to make sure our repository works correctly. We'll use:

We will test two things:
- Saving an author and retrieving them by ID
- Ensuring findById() returns empty when no author is found

`AuthorRepositoryTest.java`

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class AuthorRepositoryTest {

    private EntityManagerFactory emf;
    private EntityManager em;
    private AuthorRepository authorRepository;

    // Create EntityManagerFactory once before all tests
    @BeforeAll
    void setUpFactory() {
        emf = Persistence.createEntityManagerFactory("bookstore");
    }

    // Close EntityManagerFactory after all tests
    @AfterAll
    void tearDownFactory() {
        if (emf != null) emf.close();
    }

    // Create a new EntityManager before each test
    @BeforeEach
    void setUp() {
        em = emf.createEntityManager();
        authorRepository = new AuthorRepository(em);
    }

    // Close EntityManager after each test
    @AfterEach
    void tearDown() {
        if (em != null) em.close();
    }

    @Test
    @DisplayName("Should save and find author by ID")
    void shouldSaveAndFindAuthor() {
        // Given: a new author to be saved
        var author = new AuthorEntity()
                .setFirstName("Leo")
                .setLastName("Tolstoy")
                .setEmail("leo@books.com")
                .setAboutMe("Russian novelist");

        // When: saving the author
        Long savedId = authorRepository.save(author);

        // Then: we should be able to find the author by ID
        Optional<AuthorEntity> found = authorRepository.findById(savedId);

        assertTrue(found.isPresent(), "Author should be found by ID");
        assertEquals("Leo", found.get().getFirstName());
        assertEquals("Tolstoy", found.get().getLastName());
    }

    @Test
    @DisplayName("Should return empty if author not found")
    void shouldReturnEmptyWhenAuthorNotFound() {
        // When: searching for an ID that doesn't exist
        Optional<AuthorEntity> result = authorRepository.findById(999L);

        // Then: it should return Optional.empty()
        assertTrue(result.isEmpty(), "Should return empty Optional if not found");
    }
}

```

- `AuthorRepository` is a simple class using **EntityManager** to save and query authors.
- `AuthorRepositoryTest` verifies that saving and fetching works correctly.
- Thanks to the in-memory H2 database, tests run quickly and don’t require any setup.

---

- [Home](./../../README.md)
- [Hibernate Tutorials](./../tutorials.md)
- [Understanding Transactions in JPA and Hibernate](./2_Understanding_Transactions_in_JPA_and_Hibernate.md)