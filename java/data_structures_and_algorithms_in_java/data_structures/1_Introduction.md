# Introduction

Let's start with a simple idea. Imagine your computer's memory is a big, messy closet. You have clothes everywhere! It's hard to find your favorite shirt.

- A **data structure** is a way to organize that closet. You could use shelves, drawers, or hangers. 
Each method has good and bad points. A stack is like a stack of T-shirts (you can only get the top one), while an array is like a row of shoe boxes.
- An **algorithm** is the method you use to work with your organized clothes. It's the set of steps you follow to "`add a new pair of socks`" or "`find your blue jacket`."

Different data structures are good at different jobs. Choosing the right one is a huge part of writing fast and efficient software.

## A Quick Look at Common Structures

Here is a quick look at a few common data structures and their main features.

|Data Structure|How It Works (Simple View)|Good For|Bad For|
|:-------------|:-------------------------|:-------|:------|
|`Array`|A row of numbered boxes.|Getting an item if you know its number (index).|Searching, adding/removing in the middle, or growing.|
|`Ordered Array`|A row of mailboxes, sorted by name.|Much faster to search than a normal array (you can guess where to look).|Very slow to add or remove items (you have to shift everything); fixed size.|
|`Stack`|A pile of items; you only touch the top one.|"Undo" features. (Last-In, First-Out)|Getting an item from the bottom or middle.|
|`Queue`|A line of items; you add to the back, take from the front.|Managing tasks or requests. (First-In, First-Out)|Getting an item from the middle.|
|`Linked List`|A chain of items, each pointing to the next.|Adding or removing items in the middle.|Getting an item (you must follow the chain).|
|`Binary Tree`|A family tree (each person can have two children).|Very fast to search, add, and delete items if the tree is balanced.|Can become unbalanced (like a tall, skinny tree), making it slow.|
|`Red-Black Tree`|A "self-balancing" family tree.|Always stays balanced, so searching, adding, and deleting are always fast.|Much more complex to build and understand.|
|`2-3-4 Tree`|A family tree where a parent can have 2, 3, or 4 children.|Always balanced; works very well for storing huge amounts of data (like on a disk).|Even more complex than a red-black tree.|
|`Hash Table`|A large coat checkroom where your ticket number tells you exactly which hook your coat is on.|Extremely fast to find, add, or remove an item if you know its key (the "ticket number").|Can use a lot of memory; slow if you don't know the key; slow to delete.|
|`Heap`|A company's organization chart.|Very fast to find the most important item (like the CEO, or the highest/lowest number).|Slow to access any other items in the chart.|
|`Graph`|A map of cities and the roads connecting them.|Perfect for modeling networks and relationships (like a social network).|Some tasks, like "find the shortest path," can be slow and very complex.|

---

- [Home](./../../../README.md)
- [Java Tutorials](./../../tutorials.md)
- [Arrays](./2_Arrays.md)