## How to Use grep with Regex

A regular expression (regex) describes a search pattern. Here are the most useful pieces for day-to-day work.

> Tip on quoting: wrap your pattern in single quotes '...' so the shell doesn’t change special characters.

**Literals:** Match exact text
```bash
grep 'rack A3' notes.txt

The new server is in rack A3.
```

**Anchors:** start `^` and end `$`
```bash
grep '^2025-08-14' server.log   # lines that start with the date

2025-08-14 10:01:22 INFO  user=alice action=login
2025-08-14 10:02:05 WARN  user=bob   action=upload size=0
2025-08-14 10:03:42 ERROR user=carol action=download code=403
2025-08-14 10:05:10 INFO  user=bob   action=logout


grep 'daily\.$'   notes.txt     # lines that end with the word "daily."

Developers should review pull requests daily.
```

**Any single character:** `.`
```bash
grep 'A.ice' data.csv     # matches "Alice" and "A-ice" (any one char between A and ice)

1,Alice,92
```
**Character sets:** `[ ... ]` and ranges
```bash
grep 'user=[abc][a-z]*' server.log # user=a..., user=b..., or user=c..., followed by letters

2025-08-14 10:01:22 INFO  user=alice action=login
2025-08-14 10:02:05 WARN  user=bob   action=upload size=0
2025-08-14 10:03:42 ERROR user=carol action=download code=403
2025-08-14 10:05:10 INFO  user=bob   action=logout

grep 'user=[a][a-z]*' server.log

2025-08-14 10:01:22 INFO  user=alice action=login
```

Negation with `[^...]` (anything except):
```bash
grep 'code=[^0-9]' server.log   # code= followed by a non-digit
```

**POSIX** classes (portable and clear inside `[]`):

POSIX character classes are named sets of characters you use inside a bracket expression.
Syntax: they must be wrapped like this: `[[:digit:]]`, `[[:alpha:]]`, `[[:space:]]`.

- The outer square brackets `[...]` mean “match one character from this set.”
- The inner `[:digit:]`, `[:alpha:]`, etc., name the set.
- They’re locale-aware: `[:alpha:]` can include accented letters in your language settings.


- Common classes:
    - `[:digit:]` → digits 0–9
    - `[:alpha:]` → letters (A–Z and a–z; plus locale letters)
    - `[:alnum:]` → letters or digits
    - `[:space:]` → whitespace (space, tab, newline, etc.)
    - `[:lower:]`, `[:upper:]`, `[:punct:]`, `[:blank:]`, `[:xdigit:]` (hex), `[:print:]`, `[:graph:]`

Important: `[:digit:]` alone won’t work. It must be inside `[...]`: `**[[:digit:]]**`.

```bash
grep -E 'name,[[:alpha:]]+,[[:digit:]]+' data.csv

```


**Repeaters:** `*`, `+`, `?`, `{m,n}`

Use extended regex (`-E`) for `+`, `?`, `{m,n}`.
```bash
grep -E 'user=[a-z]+' server.log       # one or more letters
grep -E 'size=[0-9]{1,3}' server.log   # 1 to 3 digits
grep -E 'Ali(ce)?' data.csv            # "Ali" or "Alice"

```


```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```
```bash
```

---

- [HOME](./../../../README.md)
- [Linux](./../tutorials.md)
- [How to Use grep in Linux: A Practical Guide](./4_How_to_Use_grep_in_Linux_A_Practical_Guide.md)