# Data Types and Variables

Python stores **objects** (the real data), and your code uses names to refer to those objects.
Some objects can change after you create them (**mutable**), and some cannot (**immutable**).

- **Immutable:** `int`, `float`, `complex`, `str`, `tuple`, `bytes`, `frozenset`
- **Mutable:** `list`, `set`, `dict`, `bytearray`

Python is **dynamically typed** (a name can refer to any kind of object) and **strongly typed** (Python does not silently mix types that don’t fit).

## Common built-in types

| Name           | Type        | Mutable? | Examples                        |
| -------------- | ----------- | -------- | ------------------------------- |
| Boolean        | `bool`      | no       | `True`, `False`                 |
| Integer        | `int`       | no       | `42`, `1_234_567`               |
| Floating point | `float`     | no       | `0.5`, `6.02e23`                |
| Complex        | `complex`   | no       | `2+3j`, `-0.5j`                 |
| Text string    | `str`       | no       | `"hello"`, `'data'`             |
| List           | `list`      | yes      | `["red", "green", "blue"]`      |
| Tuple          | `tuple`     | no       | `(3, 5)`, `("x", "y", "z")`     |
| Bytes          | `bytes`     | no       | `b"\x00A\x7F"`                  |
| Bytearray      | `bytearray` | yes      | `bytearray([65, 66, 67])`       |
| Set            | `set`       | yes      | `{2, 3, 5}`                     |
| Frozen set     | `frozenset` | no       | `frozenset({"north", "south"})` |
| Dictionary     | `dict`      | yes      | `{"user": "Ana", "age": 30}`    |


> **NOTE:** For sets, prefer `{1, 2, 3}` over `set([1, 2, 3])` in modern code.

## Variables are names, not boxes

Think of a `name` as a sticky label. The object is the box that holds the real data.
When you do assignment, you stick the label on a box—you don’t copy the box.

```py
a = [10, 20]    # a -> that list
b = a           # b -> the same list
b.append(30)
print(a)        # [10, 20, 30]  (both names see the change)
```

To make a real copy of a list:

```py
nums = [1, 2, 3]
copy1 = nums.copy()     # shallow copy
copy2 = nums[:]         # slicing copy (also shallow)
```

If you work with nested structures and need a deep copy:

```py
import copy
deep_copy = copy.deepcopy(nums)
```

## Checking types

- `type(x)` shows the exact type.
- `isinstance(x, SomeType)` is usually better because it also works with subclasses.

```py
print(type(7) == int)        # True
print(isinstance(7, int))    # True
```

> **NOTE:** In Python, `bool` is a subclass of `int`. `isinstance(True, int)` is `True`. So be careful when you check types in numeric code.

When no names refer to an `object` anymore, Python can free that memory automatically (garbage collection). You don’t need to do it by hand.

## Booleans and “truthiness”

- `bool(x)` converts a value to `True`/`False`.

- Numbers: 
    - non-zero → `True`
    - zero → `False`
- Empty containers and None: `False`
- Non-empty strings/containers: `True`

```py
bool(7)        # True
bool(0.0)      # False
bool("")       # False
bool([1])      # True
bool(None)     # False
```

## Integers

Underscores help you read long numbers:
```py
million = 1_000_000
print(million)    # 1000000  (underscores don’t print)
```

> MISTAKE TO AVOID: `var = 1,000,000` creates a `tuple (1, 0, 0)`, not an integer. Commas separate values.

---

- [Home](./../../README.md)
- [Python Tutorials](./../tutorials.md)
- [Loops](./1_loops.md)