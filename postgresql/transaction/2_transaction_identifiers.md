# Transaction Identifiers

```info
Author      Ter-Petrosyan Hakob
```

---

Every transaction in PostgreSQL gets a special number called an **xid**. **xid** means `transaction identifier`. 
This number helps PostgreSQL keep track of transactions.

## How xid Works

- **xid as a Counter:** The **xid** is a counter that increases by one for each new transaction. It is 
    calculated with a modulo 2³¹. This means that from any current **xid**, there are 2³¹ possible future 
    transactions and 2³¹ past transactions. The **xid** counter is cyclic—it will eventually start over from low numbers.

- **Unique xid:** PostgreSQL makes sure that no two active transactions have the same **xid**. However, because the **xid** 
    counter is automatic and limited, it will eventually `wrap around`. This is called the **xid** wraparound problem.

- **Warnings and Prevention:** When the system is near a wraparound, PostgreSQL warns you. For example, you might see this message:
    ```
    WARNING: database "testdb" must be vacuumed within 177009986 transactions
    HINT: To avoid a database shutdown, execute a database-wide VACUUM in "testdb".
    ```
    This warning tells the administrator that the system needs a cleanup soon. If nothing is done, the database will shut down to protect the data.


## How VACUUM Helps

PostgreSQL uses a tool called **VACUUM** to prevent the wraparound problem. One of the main jobs of **VACUUM** is to freeze old data. 
Every table row stores the **xid** that created it in a hidden field called **xmin**. When **VACUUM** freezes a row, PostgreSQL treats that row as old—even if the **xid** counter starts again from a low number. This helps keep the order of transactions clear and safe.

### Special Values and the Freezing Mechanism

- **Starting Value for xid:** The **xid** counter starts at the special value of 3. Numbers 1 and 2 are reserved for internal system use. 
    That means no active transaction will ever have an **xid** lower than 3.

- **Old Freezing Method:** In older PostgreSQL versions, when **VACUUM** froze a row, it replaced the row’s **xmin** with 2. 
    Because 2 is lower than 3, PostgreSQL knew the row was old and could treat it as such.

- **New Freezing Method:** Now, PostgreSQL keeps the original **xmin** value. Instead of changing it, PostgreSQL uses a special 
    marker or status bit to show that the row has been frozen. This change is important because keeping the original **xmin** 
    is useful for checking history or doing a forensic analysis later.

> **NOTE:** The **xid** counter starts at 3. This means no running transaction can have an **xid** lower than 3. 
> Old versions changed the **xmin** to 2 to mark rows as frozen, but new versions use a status bit to keep the original **xmin** available.


## Virtual and Real Transaction Identifiers

PostgreSQL tries not to waste **xid** numbers. When a transaction begins, it first gets a virtual **xid**. 
A virtual **xid** is not taken from the main counter. This way, if a transaction only reads data without making any 
changes, it does not use a real **xid** number.

- **Real xid Assignment:** A real **xid** is given only when the transaction changes data 
    (for example, by using **UPDATE** or **INSERT**). For example, if you start a transaction and then cancel it:
    ```
    BEGIN;
    ROLLBACK;
    ```    
    No real **xid** is used because no change is made.

- **Checking the xid:** PostgreSQL provides two functions:
    - `txid_current()`: Always returns a real **xid**, even if the transaction is still virtual.
    - `txid_current_if_assigned()`: Returns a real **xid** only if one has been assigned. If the transaction 
        is still using a virtual **xid** (only reading), it returns **NULL**.

**Example Workflow:**

- Start a transaction and check the **xid** (no real **xid** yet):
    ```sql
        BEGIN;
        SELECT txid_current_if_assigned();
    ```
    The result is **NULL** because the transaction is still virtual.
- Do a read operation:
    
    ```sql
        SELECT count(*) FROM categories;
        SELECT txid_current_if_assigned();
    ```
    
    The result is still **NULL** since no data has changed.
- Make a data change:
    
    ```sql
        UPDATE categories SET name = UPPER(name);
        SELECT txid_current_if_assigned();
    ```   
    
    Now the function returns a number (for example, 1900). After a change, 
    both `txid_current_if_assigned()` and `txid_current()` give the same real **xid**.  

By separating virtual and real **xids**, PostgreSQL saves **xid** numbers and uses a real **xid** only when needed.    

---

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)
- [Introducing Transactions](./1_Introducing_transactions.md)
- [Multi-Version Concurrency Control](./3_Multi_Version_Concurrency_Control.md)