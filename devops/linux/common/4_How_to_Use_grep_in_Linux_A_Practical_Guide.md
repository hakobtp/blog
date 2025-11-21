# How to Use grep in Linux: A Practical Guide

`grep` searches text and shows the lines that match a pattern (a word or a regular expression). 
The name comes from global regular expression print. 
You can use it to find errors in logs, filter data, or check code quickly.

Quick commands

```bash
grep "text" file.txt          # Find lines containing "text"
grep -i "text" file.txt       # Case-insensitive match
grep -r "text" src/           # Search all files under a directory (recursively)
grep -rl "text" src/          # List filenames whose CONTENT includes "text"

find . -type f -name "*Test*"     # Filenames that CONTAIN "Test" (case-sensitive)
find . -type f -iname "*test*"    # Filenames that contain "test" (case-insensitive)

```

- `grep` searches inside files.
    - Use `grep -rl` when you want the names of files whose contents match a pattern. 
    - Use `find ... -name/-iname` when you want filenames that contain a string.

## Sample files (so you can follow along)

Create three small files in your home directory:

```bash
# 1) server.log
cat > server.log <<'EOF'
2025-08-14 10:01:22 INFO  user=alice action=login
2025-08-14 10:02:05 WARN  user=bob   action=upload size=0
2025-08-14 10:03:42 ERROR user=carol action=download code=403
2025-08-14 10:05:10 INFO  user=bob   action=logout
EOF

# 2) notes.txt
cat > notes.txt <<'EOF'
Remember to back up the database on Friday.
The new server is in rack A3.
Developers should review pull requests daily.
EOF

# 3) data.csv
cat > data.csv <<'EOF'
id,name,score
1,Alice,92
2,Bob,81
3,Caroline,92
4,Ali,70
EOF

```

## Basic usage

Find a word
```bash
grep "ERROR" server.log #Shows lines that contain ERROR.

2025-08-14 10:03:42 ERROR user=carol action=download code=403
```

Ignore case
```bash
grep -i "error" server.log #Matches ERROR, Error, error, etc.

2025-08-14 10:03:42 ERROR user=carol action=download code=403
```

Show line numbers
```bash
grep -n "upload" server.log #Helpful when you want to edit that line later.

2:2025-08-14 10:02:05 WARN  user=bob   action=upload size=0
```

Invert the match (show lines without the word)
```bash
$ grep -v "INFO" server.log #Useful to focus on warnings or errors only.

2025-08-14 10:02:05 WARN  user=bob   action=upload size=0
2025-08-14 10:03:42 ERROR user=carol action=download code=403

```

Search many files (recursive)
```bash
$ grep -r "login" . #-r searches inside subfolders too.

./server.log:2025-08-14 10:01:22 INFO  user=alice action=login
```

Only show the file names that match
```bash
$ grep -l "download" *.log #-l = list matching files, not the lines.

server.log
```

Count matches
```bash
$ grep -c "Alice" data.csv #Good for quick stats.

1
```

## Make results easier to read

Color highlights (often on by default)

```bash
grep --color=auto -n "WARN" server.log

```

Show context lines around a match

```bash
grep -nC2 "ERROR" server.log   # 2 lines before and after

1-2025-08-14 10:01:22 INFO  user=alice action=login
2-2025-08-14 10:02:05 WARN  user=bob   action=upload size=0
3:2025-08-14 10:03:42 ERROR user=carol action=download code=403
4-2025-08-14 10:05:10 INFO  user=bob   action=logout


grep -nA1 "WARN" server.log    # 1 line After

2:2025-08-14 10:02:05 WARN  user=bob   action=upload size=0
3-2025-08-14 10:03:42 ERROR user=carol action=download code=403


grep -nB1 "logout" server.log  # 1 line Before

3-2025-08-14 10:03:42 ERROR user=carol action=download code=403
4:2025-08-14 10:05:10 INFO  user=bob   action=logout
```

## Match whole words or whole lines

Whole word

```bash
grep -w "Alice" data.csv #This avoids matching “Alicea” or “Malice”.

1,Alice,92
```

Whole line

```bash
grep -x "Developers should review pull requests daily." notes.txt

Developers should review pull requests daily.
```

## Fast literal search (no regex)

Sometimes you want to search exact text and skip regex rules.

```bash
grep -F "user=bob   action=upload" server.log

2025-08-14 10:02:05 WARN  user=bob   action=upload size=0
```

`-F` treats the pattern as a fixed string (faster and simpler).

### Short cheat sheet

```bash
grep "text" file        # simple search
grep -i "text" file     # ignore case
grep -n "text" file     # show line numbers
grep -v "text" file     # invert match (lines without)
grep -r "text" dir/     # recursive
grep -l "text" files*   # only filenames
grep -c "text" file     # count
grep -w "word" file     # whole word
grep -x "line" file     # whole line
grep -F "a*b" file      # fixed string (no regex)
grep -E "a|b" file      # extended regex
grep -A2 -B2 "x" file   # show context
```

### Performance & safety tips
- **Quote your pattern:** use single quotes 'pattern' so the shell doesn’t expand `*` or `?`.
- **Use:** `-F` for plain text searches: faster and avoids regex confusion.
- **Binary files:** add `-I` to skip or `-a` to treat them as text.
- **Locale:** if you need pure byte speed, `LC_ALL=C grep ...` can be faster (advanced).
- **Color:** `--color=auto` helps you see matches quickly.


### File filters when searching directories

Include or exclude certain files as you recurse:

```bash
# Only search .log files in logs/
grep -r --include='*.log' 'ERROR' logs/

# Exclude big folders and temp files
grep -r --exclude-dir='node_modules' --exclude='*.tmp' -n 'TODO' .

# Multiple includes (repeat the flag)
grep -r --include='*.{js,ts}' --include='*.tsx' -n '\buse strict\b' src/
# (Brace expansion may be a shell feature; when unsure, repeat --include)
```

On many systems: `-R` follows symlinks, `-r` usually doesn’t. Check man grep for your platform.

---

- [HOME](./../../../README.md)
- [DevOps](./../../tutorials.md)
- [System Info and Processes](./3_System_Info_and_Processes.md)
- [How to Use grep with Regex](./5_How_to_Use_grep_with_Regex.md)