# Understanding Locks

When I first started learning about databases, I believed there were only two kinds of locks: **shared** and **exclusive**.

- A **shared lock** (or read lock) means that many people can read the same data at the same time.
- An **exclusive lock** (or write lock) means that only one process can make changes — everyone else must wait.

It sounded simple. But when I started exploring PostgreSQL, I discovered that the real picture is far more detailed and powerful. 
PostgreSQL has five lock categories and over a dozen individual lock types, each with its own purpose and behavior.

In this post, we’ll explore:

- How PostgreSQL manages table and row locks.
- What commands take which locks.
- Which locks block others.
- And finally, why understanding this helps you write better, faster, and safer SQL.

Let’s begin our journey into PostgreSQL’s world of locks.

## Table Locks — Controlling Access to Tables

If you asked me a few years ago what a table lock was, I might have said: “It’s a lock on a table that stops anyone else from using it.”

That sounds logical but isn’t true in PostgreSQL. In reality, PostgreSQL has eight table lock types, and a transaction can hold more than one lock on the same table. Some locks allow other operations to continue; others block everything.

You can check table locks by running:

---

- [Home](./../../README.md)
- [PostgreSql Tutorials](./../tutorials.md)