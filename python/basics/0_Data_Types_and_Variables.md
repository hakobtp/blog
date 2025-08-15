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


## Basic operators

- `+` `-` `*` — add, subtract, multiply
- `/` — true division → always a float
- `//` — floor division → rounds down to an integer result
- `%` — remainder (modulo)
- `**` — exponent

```py
7 / 2     # 3.5
7 // 2    # 3
7 % 2     # 1
3 ** 4    # 81
```

With negatives (good to know):

```py
-7 // 2   # -4  (rounds down)
-7 % 2    # 1   (because: (-7) = 2*(-4) + 1)
```

Get quotient and remainder together:

```py
divmod(19, 5)  # (3, 4)
```

Exponent edge cases:
```py
0 ** 3   # 0
0 ** 0   # 1   (by Python’s rules)
2 ** 3.0 # 8.0 (mixing int and float -> float)
```

## Number bases

You can write integers in other bases with prefixes:

- Binary: 0b1010
- Octal: 0o12
- Hex: 0xFF

Convert a decimal number to strings in other bases:

```py
n = 255
bin(n)   # '0b11111111'
oct(n)   # '0o377'
hex(n)   # '0xFF'
```

Convert from a string in some base to an integer:

```py
int("1010", 2)   # 10
int("ff", 16)    # 255
int("10", 22)    # 22  (bases 2..36 are allowed)
```

Also handy:
```py
chr(65)   # 'A'
ord('A')  # 65
```

## Type conversions

```py
int(True)      # 1
int(False)     # 0
int(98.6)      # 98   (truncates toward zero)
int("99")      # 99
int("-23")     # -23
```

> **CORRECTION:** `int("1_000_000")` raises `ValueError`. Underscores work in code literals (`1_000_000`), not inside strings.

If you must parse a string with underscores:
```py
int("1_000_000".replace("_", ""))  # 1000000
```

Invalid numeric strings cause `ValueError`:

```py
int("")                      # ValueError
int("99 bottles of milk")    # ValueError
```

Floats:
```py
float(98)         # 98.0
float("98.6")     # 98.6
float("1.0e4")    # 10000.0
```

Mixing ints and floats produces floats:
```py
4 + 7.0      # 11.0
True + 2     # 3  (True behaves like 1)
False + 5.0  # 5.0
```

**NOTE:** Floating-point numbers are approximate. For money, consider `decimal.Decimal`.


## Reassigning names vs copying

Reassigning just moves the name to a new object; it does not change the old object.

```py
x = 5
y = x
x = 29
print(y, x)   # 5 29
```

For mutables, both names see changes unless you copy:

```py
a = ["A", "B"]
b = a
a[0] = "Z"
print(b)      # ['Z', 'B']  (same list)

b = a.copy()  # now b is a separate list
a[1] = "Y"
print(a)      # ['Z', 'Y']
print(b)      # ['Z', 'B']
```

## Writing long lines

You can break long code over multiple lines. Two safe methods:

Use parentheses/brackets/braces (preferred):
```py
total = (
    1 +
    2 +
    3 +
    4
)
```

Use a backslash `\` (works, but easier to make mistakes if you add spaces after it):

```py
total = 1 + \
        2 + \
        3 + \
        4
```

## Python: strong + dynamic

Types belong to objects, and checks happen **at runtime**.  

```py
"3" + 3        # TypeError (no silent coercion from str to int)
int("3") + 3   #  6 (explicit conversion)
len(3)         #  TypeError
3 + 4.0        #  7.0 (numeric promotion is allowed)
True + 2       #  3  (bool is a subclass of int in Python)

```

Python is dynamically and strongly typed,” not “weakly typed, because Python refuses unsafe implicit conversions.

You can add type hints and use a checker like mypy or Pyright for early (static-like) feedback, while runtime stays dynamic.

```py
def greet(name: str) -> str:
    return "Hello, " + name
```

This runs with or without hints, but a checker can warn you if you call `greet(123)`.

---

- [Home](./../../README.md)
- [Python Tutorials](./../tutorials.md)
- [If Statement](./01_If_Statement.md)