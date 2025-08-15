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

---

- [Home](./../../README.md)
- [Python Tutorials](./../tutorials.md)