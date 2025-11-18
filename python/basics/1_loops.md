# Loops

In this guide you’ll learn how to repeat actions in Python with `while` and `for`. 
You’ll also learn how `break`, `continue`, and the special `else` on loops work. The examples are simple and practical.

## Why loops matter

Loops help you run the same code many times. They save time, reduce mistakes, and make programs clear. 
You can use loops to process a list of items, read user input until a keyword, or search for something inside data.

## When to use each loop

- Use `for` when you already have a collection (a list, a string, a file) or a known range of numbers. You say: “Go through each item.”
- Use `while` when you want to repeat until a condition changes. You say: “Keep going while this is true.”

> **TIP:** If you can count or iterate through items, prefer `for`. If you are waiting for a condition (e.g., user types stop), prefer `while`.

## while: repeat while a condition is true

Structure:

```py
while <condition>:
    # repeated code
```

The loop stops when the condition becomes `False`.

**Example A** — Keep doubling until the number is large enough

```py
n = 6
while n <= 100:
    print(n)
    n *= 2
print("Done")

```
What happens? Prints `6`, `12`, `24`, `48`, `96` and then stops. We updated `n` each time; this prevents an infinite loop.


**Example B** — Sentinel loop with break and continue

Ask for words until the user types stop. Ignore very short words.

```py
while True:
    word = input("Word (type 'stop' to finish): ")
    if word == "stop":
        break              # leave the loop completely
    if len(word) < 3:
        continue           # skip the rest of this round
    print("Accepted:", word)
```

The `else` block runs only if the loop ended normally (condition became `False`) and no `break` was used.

```py
numbers = [3, 5, 12, 7]
index = 0

while index < len(numbers):
    if numbers[index] < 0:
        print("Found a negative:", numbers[index])
        break
    index += 1
else:  # executed only if we never broke out
    print("No negatives found")
```

> **IDEA:** Think of `else` as a “break checker.” If you never `break`, `else` runs.

## for: go through items in a sequence

Structure:

```py
for item in <iterable>:
    # use item
```

**Example A** — Shopping list with continue and break

```py
items = ["bread", "apples", "sugar", "eggs", "tea"]

for item in items:
    if item == "sugar":
        continue              # skip sugar
    print("Buy:", item)
    if item == "eggs":
        print("Got eggs — stopping early.")
        break                 # stop the whole loop
```

`for ... else`: run only if loop didn’t break

```py
word = "piano"
for ch in word:
    if ch == "z":
        print("Found a 'z'.")
        break
else:
    print("No 'z' found.")
```    
 
> **USE CASE:** Searching. If you don’t find what you want (no break), the else part confirms that.

## range(): number streams for counting

`range()` produces numbers on demand (it doesn’t build a big list by default).
- `range(stop)` → 0 up to (but not including) stop
- `range(start, stop)`
- `range(start, stop, step)` (step can be negative)

```py
for i in range(1, 4):   # 1, 2, 3
    print(i)

print(list(range(5)))            # [0, 1, 2, 3, 4]
print(list(range(10, 0, -2)))    # [10, 8, 6, 4, 2]
```

> **REMEMBER:** the stop value is excluded.

## Helpful patterns

Get index and value with `enumerate`
```py
letters = "data"
for pos, ch in enumerate(letters):
    print(pos, ch)
```

Nested loops (a small grid)
```py
for r in range(2):
    for c in range(3):
        print(f"({r},{c})")
```

Simple countdown
```py
for seconds in range(3, 0, -1):
    print(seconds)
print("Lift-off!")
```

## enumerate

`enumerate()` is a built-in Python function that lets you loop over a sequence and get both the `index` and the `value` at the same time.

```py
enumerate(iterable, start=0)
```

- `iterable`: something you can loop over (list, string, etc.)
- `start`: the first index to use (default is `0`)
- What it returns: an **iterator** that gives pairs `(index, value)`.

List with default indexing (starts at 0)

```py
fruits = ["apple", "banana", "pear"]
for i, fruit in enumerate(fruits):
    print(i, fruit)
# 0 apple
# 1 banana
# 2 pear
```

Start counting from 1 (useful for line numbers)

```py
lines = ["first", "second", "third"]
for line_no, text in enumerate(lines, start=1):
    print(f"Line {line_no}: {text}")
# Line 1: first
# Line 2: second
# Line 3: third
```

String characters with positions

```py
for pos, ch in enumerate("data"):
    print(pos, ch)
# 0 d
# 1 a
# 2 t
# 3 a
```

See the pairs directly

```py
list(enumerate("cat"))
# [(0, 'c'), (1, 'a'), (2, 't')]
```

Why use `enumerate()` instead of `range(len(...))`?

This:
```py
for i in range(len(fruits)):
    print(i, fruits[i])
```

works, but is less readable and easier to make mistakes with. Prefer:

```py
for i, fruit in enumerate(fruits):
    print(i, fruit)
```

- if you don’t need the `index`, don’t use `enumerate()`—just write `for x in seq:`.
- Avoid changing the list while iterating. If needed, loop over a copy.
- **Remember:** `enumerate()` returns an iterator; convert to a list only if you really need to.

## The walrus operator := (Python 3.8+)

The walrus operator **assigns and returns** a value in one expression.
Use it when it **makes code clearer**, not just shorter.

```py
# Keep reading words until the user enters a blank line
while (word := input("Word (blank to stop): ")) != "":
    print(f"You typed: {word}")
```

```py
numbers = [12, -3, 7, 0, 15]
for n in numbers:
    if (gap := 10 - n) >= 0:
        print(f"{n} fits; {gap} space left.")
```


## Common mistakes (and quick fixes)

- Forgetting to update the variable in `while` → infinite loop. Always change the variable inside the loop.
- Confusing `break` and `continue` → `break` leaves the loop; `continue` skips to the next round.
- Using `range(len(seq))` when you only need values → iterate directly: `for x in seq`:. Use `enumerate` if you also need the index.
- Misreading `for/while ... else` → `else` runs only if there was no `break`.
- Off-by-one with `range()` → remember that `stop` is not included.
- Changing a list while iterating over it → iterate over a copy or build a new list.


## Recap
- **Loop:** Code that runs again and again.
- **Iteration:** One pass through the loop.
- **Condition:** A test that is True/False.
- **Counter:** A number you change each round (e.g., i += 1).
- **Sentinel value:** A special word/number that tells the loop to stop (e.g., "stop").
- **Iterable:** Something you can loop over (list, string, range).
- **Iterator:** An object that gives the next item each time.
- **break:** Exit the loop.
- **continue:** Skip the rest of this round.
- **range():** Produces numbers for counting.
- **Nested loop:** A loop inside another loop.

---

- [Home](./../../README.md)
- [Python Tutorials](./../tutorials.md)
- [If Statement](./01_If_Statement.md)