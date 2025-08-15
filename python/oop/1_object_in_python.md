# Objects in Python

In Python, **everything is an object** — numbers, strings, lists, dictionaries, even functions.
You might not always see it because Python hides most of the technical details behind its simple syntax.

For example:
```py
score = 10
```

Here’s what actually happens:

- Python creates an object of type `int` with the value `10`.
- The name `score` is linked to that object.

You don’t normally need to worry about how Python builds objects unless you want to:
- Create your own objects.
- Change the way existing ones behave.

## What exactly is an object?

An object is a container that can hold:
- **Data** → stored in variables called attributes.
- **Code** → stored in functions called methods.

You can think of:
- Objects as nouns (things).
- Methods as verbs (actions).

For example:
- A `Book` object might have `title` and `pages` as attributes.
- It might have `open()` and `read_page()` as methods.

Every object is created from a class, which acts as a blueprint.

## Objects are everywhere

Examples of built-in Python objects:
- `42` → integer object from the `int` class.
- `"hello"` → string object from the `str` class, with methods like `.upper()` and `.replace()`.
- `[1, 2, 3]` → list object from the `list` class, with methods like `.append()` and `.sort()`.

## Multiple objects, different values

Unlike modules (which are single files), you can have many objects of the same type at the same time, each with its own attributes.

For example, you can have two different book objects:

```py
class Book:
    pass

book1 = Book()
book2 = Book()
```

This is the smallest possible class: it does nothing yet. The `pass` statement means “do nothing” and is only there so Python accepts the empty class.

If you print an object without a custom display method, Python shows its `type` and a `memory address`:

```py
class Dog:
    pass

buddy = Dog()
rocky = Dog()

print(buddy)  # <__main__.Dog object at 0x7f21a8b2fa90>
print(rocky)  # <__main__.Dog object at 0x7f21a8b2fcd0>

```

These memory addresses change every time you run the code.

## Adding attributes to objects

You can add attributes to objects after creating them:

```py
buddy.name = "Buddy"
buddy.age = 4
```

Objects can also refer to other objects as attributes:

```py
buddy.friend = rocky
```

If you try to access an attribute that doesn’t exist, Python raises an `AttributeError`:

```py
print(buddy.friend.name)
# AttributeError: 'Dog' object has no attribute 'name'
```

We can fix this by giving the `friend` object a `name`:

```py
buddy.friend.name = "Rocky"
print(buddy.friend.name)  # Rocky
```

When programmers talk about “attributes,” they usually mean object attributes.
There are also class attributes, which we’ll discuss later.

## Setting attributes at creation time

If you want an object to have attributes right from the start, use Python’s initializer method `__init__()`:

```py
class Dog:
    def __init__(self, name, age):
        self.name = name
        self.age = age
```

The `self` argument specifies that it refers to the individual object itself. When you define `__init__()` in a class definition, its first parameter should be named `self` . Although `self` is not a reserved word in Python, it’s common usage.

Now we can pass values when creating a new object:

```py
puppy = Dog("Max", 2)
print(puppy.name)  # Max
print(puppy.age)   # 2
```

### How it works step-by-step

When you write:
```py
puppy = Dog("Max", 2)
```

Python:
- Finds the `Dog` class.
- Creates a new empty object in memory.
- Calls `__init__()` with:
    - The new object as `self`.
    - `"Max"` as `name`.
    - `2` as `age`.
- Stores these values in the object.
- Returns the finished object and links it to the name `puppy`.

This new object is like any other object in Python. You can use it as an element of a list, tuple, dictionary, or set. You can pass it to a function as an argument, or return it as a result.

You don’t have to put an `__init__()` method in every class.
We use it only when we want to give an object special settings or values that make it different from other objects of the same class.
In some programming languages, this is called a “constructor,” but in Python the object is already created before `__init__()` runs.
You can think of `__init__()` as a setup step that prepares the new object.

## Why use objects?

Objects make your code:
- **Organized** → keep data and actions together.
- **Reusable** → one class can produce many different objects.
- **Clear** → no need for messy lists of separate variables.

Instead of:

```py
names = ["Buddy", "Rocky"]
ages = [4, 6]
```

You can do:

```py
dogs = [
    Dog("Buddy", 4),
    Dog("Rocky", 6)
]

```

Now each dog has both its name and age stored together.

## Recap

- Everything in Python is an object.
- Objects have attributes (data) and methods (actions).
- A class is the blueprint; an object is an instance of it.
- `__init__()` lets you set attributes during creation.
- Missing attributes cause AttributeError.

---

- [Home](./../../README.md)
- [Python Tutorials](./../tutorials.md)