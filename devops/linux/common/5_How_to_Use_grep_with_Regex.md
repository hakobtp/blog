## How to Use grep with Regex

A regular expression (regex) describes a search pattern. Here are the most useful pieces for day-to-day work.

> Tip on quoting: wrap your pattern in single quotes `'...'` so the shell doesn’t change special characters.

**Literals:** Match exact text
```bash
grep 'rack A3' notes.txt
```

**Anchors:** start `^` and end `$`
```bash
grep '^2025-08-14' server.log   # lines that start with the date
grep 'daily\.$'   notes.txt     # lines that end with the word "daily."
```

**Any single character:** `.`
```bash
grep 'A.ice' data.csv     # matches "Alice" and "A-ice" (any one char between A and ice)
```

**Character sets:** `[ ... ]` and ranges
```bash
grep 'user=[abc][a-z]*' server.log # user=a..., user=b..., or user=c..., followed by letters
```

Negation with `[^...]` (anything except):
```bash
grep 'code=[^0-9]' server.log   # code= followed by a non-digit
```

## POSIX classes

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

Important: `[:digit:]` alone won’t work. It must be inside `[...]`: `[[:digit:]]`.

```bash
cat data.csv 

id,name,score
1,Alice,92
2,Bob,81
3,Caroline,92
4,Ali,70
5,O'Neil,77
6,Jean-Luc Picard!,99
6,Jean-Luc Picard,99
```

**Pattern 1:**
```bash
grep -E '^[[:digit:]]+,[[:alpha:]]+,[[:digit:]]+$' data.csv

1,Alice,92
2,Bob,81
3,Caroline,92
4,Ali,70
```

Goal: match lines like `123,Alice,92`

- `^` — start of line (anchor).
- `[[:digit:]]+` — one or more digits → the ID field.
- `,` — a literal comma (delimiter).
- `[[:alpha:]]+` — one or more letters only (A–Z, a–z; plus locale letters) → the Name field with no spaces or hyphens (e.g., Alice, Bob).
- `,` — comma.
- `[[:digit:]]+` — one or more digits → the Score field.
- `$` — end of line (anchor).

**Doesn’t match:** 1,Ali ce,92 (space), 1,Mary-Jane,92 (hyphen), header id,name,score (no leading digits).

**Pattern 2:**
```bash
grep -E '^[[:digit:]]+,[[:alpha:][:space:]-]+,[[:digit:]]+$' data.csv

1,Alice,92
2,Bob,81
3,Caroline,92
4,Ali,70
6,Jean-Luc Picard,99
```

Goal: allow multi-word or hyphenated names: `123,Mary Jane-Smith,92`

What changed
- Middle field: `[[:alpha:][:space:]-]+` one or more of:
    - `[:alpha:]` — letters,
    - `[:space:]` — whitespace (space, tab, etc.),
    - `-` — a literal hyphen

So names like `“Alice Smith”` or `“Mary-Jane”` now match.

**Doesn’t match:** 5,O'Neil,77 (apostrophe not allowed), 6,Jean-Luc Picard!,99 (! not allowed)

- Key differences (at a glance)
    - **Pattern 1:** Name = letters only
    - **Pattern 2:** Name = letters + spaces + hyphens
- Both require:
    - **Field 1:** `digits` **Field 2:** `alpha` Field 3: `digits`
    - Fields separated by commas, line fully matched due to `^` and `$`.    


**Allow apostrophes in names:**
```bash
grep -E '^[[:digit:]]+,[[:alpha:][:space:]'\''-]+,[[:digit:]]+$' data.csv

1,Alice,92
2,Bob,81
3,Caroline,92
4,Ali,70
5,O'Neil,77
6,Jean-Luc Picard,99
```

(Note the escaped `'` inside single quotes.)

**Allow only space (not tabs) between words:**

```bash
grep -E '^[[:digit:]]+,[[:alpha:] -]+,[[:digit:]]+$' data.csv
```

(That middle class includes ` ` and `-`, but not `[:space:]`.)

if you need non-ASCII letters (e.g., accents), ensure your locale is set (e.g., `LC_ALL=en_US.UTF-8`), because `[:alpha:]` is locale-aware.

---

- [HOME](./../../../README.md)
- [Linux](./../tutorials.md)
- [How to Use grep in Linux: A Practical Guide](./4_How_to_Use_grep_in_Linux_A_Practical_Guide.md)