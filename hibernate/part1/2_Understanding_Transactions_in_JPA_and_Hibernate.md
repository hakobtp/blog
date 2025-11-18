# Understanding Transactions in JPA and Hibernate

```info
Author      Ter-Petrosyan Hakob
```
---

A transaction is a group of operations that should run together as one unit.
If all operations succeed, the transaction is committed. If even one fails, the transaction is rolled back, and all changes are undone.

This helps your application keep the data safe and consistent.

You can start and manage a transaction in JPA like this:

```java
entityManager.getTransaction().begin();   // Start transaction
// ... perform operations like persist, merge, etc.
entityManager.getTransaction().commit();  // Save changes
// or
entityManager.getTransaction().rollback(); // Undo changes if something goes wrong

```

**Why Are Transactions Important?**

Enterprises rely on transactions every day—for things like:
- Bank transfers
- Online purchases
- Sending data to other services

These operations may involve:
- Writing to one or more databases
- Sending messages to another system
- Calling external APIs

All these actions should happen together. If one fails, none should be applied.

**What Makes a Good Transaction? (ACID Properties)**

Transactions follow four main rules called ACID:

| Property     | Description |
|--------------|-------------|
| **Atomicity** | All steps must succeed or none should. For example, if you're transferring money, you should debit one account and credit another in the same unit. |
| **Consistency** | The database must stay valid. If your database has rules (like unique IDs or foreign keys), those rules should never break. |
| **Isolation** | Other parts of the system can’t see changes until the transaction is done. They see either the data before or after—not during. |
| **Durability** | Once committed, the changes stay, even after a crash or restart. |


**Example: Bank Transfer**

Imagine you're moving $100 from your Savings account to your Checking account. That transaction may involve:

1. Debiting the **Savings** account<br>
2. Crediting the **Checking** account<br>
3. Logging the transfer in a table

All of these steps must happen **together**.
If one fails, the whole transaction should roll back. **Why?** Because you don’t want to lose money or create incorrect records.


## Managing Transactions with Hibernate

Let’s go one step deeper. Suppose we’re building a simple transfer system between two users — and we want to make sure it behaves atomically, like a bank.

We’ll write an example where:
- Each user has a balance
- One user sends money to another
- If something goes wrong (like negative balance), the transaction rolls back

**Step 1 Define the Entity:** Let’s use a simple `UserAccountEntity`:

```java
@Entity
@Getter
@Setter
@Accessors(chain = true)
@Table(name = "accounts")
public class UserAccountEntity {

    @Id
    @GeneratedValue
    private Long id;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private double balance;
}
```

**Step 2 Write the Repository:** This repository will include a transfer() method wrapped in a manual transaction.

```java
@RequiredArgsConstructor
public class AccountRepository {

    private final EntityManager em;

    public void transfer(Long fromId, Long toId, double amount) {
        EntityTransaction tx = em.getTransaction();

        try {
            tx.begin();

            UserAccountEntity from = em.find(UserAccountEntity.class, fromId);
            UserAccountEntity to = em.find(UserAccountEntity.class, toId);

            if (from == null || to == null) {
                throw new IllegalArgumentException("Account not found");
            }

            if (from.getBalance() < amount) {
                throw new IllegalArgumentException("Insufficient funds");
            }

            // Update balances
            from.setBalance(from.getBalance() - amount);
            to.setBalance(to.getBalance() + amount);

            tx.commit(); // All operations succeed — commit
        } catch (Exception e) {
            tx.rollback(); // Something went wrong — rollback
            throw new RuntimeException("Transfer failed", e);
        }
    }

    public void save(UserAccountEntity account) {
        em.getTransaction().begin();
        em.persist(account);
        em.getTransaction().commit();
    }

    public Optional<UserAccountEntity> findById(Long id) {
        return Optional.ofNullable(em.find(UserAccountEntity.class, id));
    }
}
```

What Happens Behind the Scenes?

- `em.getTransaction().begin()` starts the transaction.
- If any part of the transfer fails (e.g. account not found, not enough balance), we call `tx.rollback()`.
- Otherwise, if all goes well, we `commit()` the changes.
- If committed, Hibernate flushes the updated state to the database.

**Step 3: AccountRepositoryTest**

```java
@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class AccountRepositoryTest {

    private EntityManagerFactory emf;
    private EntityManager em;
    private AccountRepository accountRepository;

    @BeforeAll
    void initFactory() {
        emf = Persistence.createEntityManagerFactory("bookstore");
    }

    @AfterAll
    void closeFactory() {
        if (emf != null) emf.close();
    }

    @BeforeEach
    void init() {
        em = emf.createEntityManager();
        accountRepository = new AccountRepository(em);
    }

    @AfterEach
    void tearDown() {
        if (em != null) em.close();
    }

    @Test
    @DisplayName("Should transfer money between accounts")
    void shouldTransferMoney() {
        // Given
        var alice = new UserAccountEntity().setName("Alice").setBalance(500);
        var bob = new UserAccountEntity().setName("Bob").setBalance(300);

        accountRepository.save(alice);
        accountRepository.save(bob);

        // When
        accountRepository.transfer(alice.getId(), bob.getId(), 200);

        // Then
        var updatedAlice = accountRepository.findById(alice.getId()).orElseThrow();
        var updatedBob = accountRepository.findById(bob.getId()).orElseThrow();

        assertEquals(300, updatedAlice.getBalance(), 0.01);
        assertEquals(500, updatedBob.getBalance(), 0.01);
    }

    @Test
    @DisplayName("Should rollback if insufficient funds")
    void shouldRollbackTransferIfInsufficientFunds() {
        // Given
        var alice = new UserAccountEntity().setName("Alice").setBalance(100);
        var bob = new UserAccountEntity().setName("Bob").setBalance(300);

        accountRepository.save(alice);
        accountRepository.save(bob);

        // When & Then
        assertThrows(RuntimeException.class, () ->
                accountRepository.transfer(alice.getId(), bob.getId(), 200));

        var updatedAlice = accountRepository.findById(alice.getId()).orElseThrow();
        var updatedBob = accountRepository.findById(bob.getId()).orElseThrow();

        // Ensure no balances changed (transaction rolled back)
        assertEquals(100, updatedAlice.getBalance(), 0.01);
        assertEquals(300, updatedBob.getBalance(), 0.01);
    }
}
```

**Key Lessons:**

| Concept            | Description                                                           |
|--------------------|------------------------------------------------------------------------|
| **Transaction block** | You control when a transaction starts, commits, or rolls back           |
| **Consistency**       | All balances are changed together, or not at all                         |
| **Rollback safety**   | Errors don’t corrupt your database                                       |


---

- [Home](./../../README.md)
- [Hibernate Tutorials](./../tutorials.md)
- [Setting Up Hibernate with Maven (Step-by-Step Guide)](./1_maven_project_setup.md)
- [Inheritance](./3_Inheritance.md)