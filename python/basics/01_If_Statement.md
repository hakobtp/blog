# If Statement

An `if` statement lets your program make a decision. It checks a condition.
- If the condition is `True`, Python runs a block of code.
- If it’s `False`, Python can skip it or run a different block (with else).

```py
is_raining = True

if is_raining:
    print("Take an umbrella.")
else:
    print("Enjoy the sunshine!")
```

Use `elif` (“else if”) to test another condition when the first one is `False`.

```py
light = "yellow"

if light == "red":
    print("Stop")
elif light == "yellow":
    print("Slow down")
else:
    print("Go")
```

> **NOTE:** `==` tests equality. `=` assigns a value. Don’t mix them up.

## Boolean operators: and, or, not

You can combine conditions.
```py
age = 16
with_parent = True

if age < 18 and with_parent:
    print("You can enter with a parent.")

role = "moderator"
if role == "admin" or role == "moderator":
    print("Access granted.")

is_weekend = False
if not is_weekend:
    print("It’s a weekday.")
```

A clearer pattern for many options use `in`. Instead of chaining many `or`s, use membership with `in` / `not in`.

```py
role = "moderator"
elevated = {"admin", "moderator", "owner"}

if role in elevated:
    print("Access granted.")
```

`in` also works with lists, tuples, strings, and dicts (for dicts it checks keys):

```py
code = "AM"
allowed = ["AM", "GE", "FR"]         # list
print(code in allowed)               # True

vowels = ("a", "e", "i", "o", "u")   # tuple
print("o" in vowels)                 # True

letters = {"a", "b", "c"}            # set
print("d" in letters)                # False

fruit_by_letter = {"a": "apple", "b": "banana"}  # dict
print("a" in fruit_by_letter)        # True  (checks keys, not values)

word = "notebook"                    # string
print("note" in word)                # True  (substring)
```

## Truthy and falsy values

`if` doesn’t only use `True`/`False`. Many values are truthy or falsy. 

Falsy values (treated as `False`):
- `False`, `None`, `0`, `0.0`
- empty string `""`
- empty list `[]`, empty tuple `()`
- empty dict `{}`, empty set `set()`

Anything else is truthy.

```py
items = []
if items:
    print("We have items.")
else:
    print("The list is empty.")   # runs
```

## Chained comparisons

Python lets you chain comparisons for readability.

```py
temperature = 23
if 18 <= temperature <= 25:
    print("Comfortable range.")
```

This is cleaner than: `if temperature >= 18 and temperature <= 25:`.

## Common mistakes (and quick fixes)

- Using `=` instead of `==` in conditions → `if x == 5`:
- Forgetting the colon `:` → `if condition:`
- Bad indentation → indent the block under `if`, `elif`, `else`.
- Comparing to `None` with `==` → prefer is: `if x is None:`
- Long chains of `or` → prefer in, e.g., `if ext in {".py", ".js"}:`

## The walrus operator := (Python 3.8+)

The walrus operator **assigns and returns** a value in one expression.
Use it when it **makes code clearer**, not just shorter.

```py
limit = 100
msg = "OK" * 42  # length 84

if (left := limit - len(msg)) >= 0:
    print(f"Fits: {left} characters left")
else:
    print(f"Too long by {-left}")
```

Why the parentheses? `(left := limit - len(msg)) >= 0` first assigns to `left`, then compares.
Without parentheses, you might accidentally compare before assignment.


---

- [Home](./../../README.md)
- [Python Tutorials](./../tutorials.md)
- [Data Types and Variables](./0_Data_Types_and_Variables.md)
- [Loops](./1_loops.md)