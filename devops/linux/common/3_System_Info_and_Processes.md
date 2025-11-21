# System Info and Processes

This guide helps you check **system information** and **running processes** on Linux. 
Each command includes: what it does, how to read the output, and small tips for real work.

Quick map of commands:

| Command      | Purpose                        | Quick memory                    |
| ------------ | ------------------------------ | ------------------------------- |
| `uname -a`   | Kernel and system version      | “Who am I?” (system identity)   |
| `df -h`      | Disk space per filesystem      | “How full are my disks?”        |
| `free -h`    | Memory and swap usage          | “How much RAM is really free?”  |
| `uptime`     | Time since boot + load average | “How busy is the system?”       |
| `ps aux`     | List all processes             | “What is running now?”          |
| `top`        | Live view of CPU/RAM/processes | “What is eating resources now?” |
| `kill <PID>` | Send a signal to a process     | “Please stop (politely first)!” |


---

**uname -a** — Kernel and system version

What it shows 
- System identity: kernel name and version, machine type (e.g., x86_64), and more.

```bash
uname -a

Linux myhost 6.8.0-39-generic #42-Ubuntu SMP PREEMPT_DYNAMIC x86_64 GNU/Linux
```

How to read
- **Linux:** kernel name.
- **myhost:** your machine (node) name.
- **6.8.0-39-generic:** kernel release (version).
- **#42-Ubuntu …:** build info.
- **x86_64:** architecture (64-bit).
- **GNU/Linux:** OS family.

Why it matters
- When you report a bug or install drivers, kernel and arch are important.

Tip

- `uname -r` → only the kernel release
- `uname -m` → only the machine hardware name (e.g., x86_64, aarch64)

---

**df -h** — Disk space

What it shows
- How much space is used and available on each mounted filesystem.

```
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3       200G  150G   40G  79% /
tmpfs            16G   60M   16G   1% /run
/dev/sdb1       500G  320G  161G  67% /data
```

Key columns
- **Filesystem:** device or volume name
- **Size /Used/Avail:** total, used, and free space
- **Use%:** percent used
- **Mounted on:** where it is attached in your directory tree

Common questions
- “What is `tmpfs`?” → A memory-backed filesystem (temporary).

Tips

- `df -hT` adds `type` (e.g., ext4, xfs).
- `df -h /home` shows only the filesystem for /home.

---

**free -h** — Memory usage

What it shows
- How your RAM and swap are used.

```bash
free -h
              total   used   free  shared  buff/cache  available
Mem:            16G     3G     1G    300M         12G        13G
Swap:            4G   256M     3.8G
```

Important ideas (simple but deep)

- Linux uses free RAM as cache to make programs faster.
- Because of cache, the “used” number looks big, but much of it is cache and can be freed.
- “available” is the best quick number to ask: “How much memory can apps use right now without swapping?”


Columns
- **total:** total RAM
- **used:** used now (includes cache)
- **free:** completely unused RAM (usually small on purpose)
- **shared:** RAM shared by tmpfs/shmem
- **buff/cache:** file cache + buffers
- **available:** safe estimate of RAM apps can still use

**Swap** Used when RAM is not enough. If Swap used grows and the system is slow, you may not have enough RAM for your workload.

---

**uptime** — Time since boot & load average

What it shows
- How long the system has been up
- Number of logged-in users
- Load average for the last 1, 5, 15 minutes

```bash
uptime

11:20:03 up 5 days,  3:12,  2 users,  load average: 0.85, 1.10, 0.90
```

Load average (very important)

- Roughly: how many processes are running or waiting for CPU/IO.
- Compare it with the number of CPU cores.
    - If you have 4 cores, a load around 4.0 means all cores are busy.
    - If the load is much higher than core count for a long time, the system is overloaded.

```bash
nproc    # shows number of CPU cores
```

### Processes 101

A process is a running program. Each process has a `PID` (process ID).
There is often a parent process (`PPID`) that started it.

Why you care:
- To find what is using CPU or memory
- To stop a hung or misbehaving program
- To understand system load

--- 

**ps aux** — List all processes

What the options mean

- **a:** show processes from all users
- **u:** show in a user-friendly format (with usernames)
- **x:** include processes without a terminal

```bash
$ ps aux | head
USER     PID  %CPU %MEM    VSZ   RSS TTY   STAT START   TIME COMMAND
root       1   0.0  0.1 168324 11000 ?     Ss   Aug09   0:10 /sbin/init
hakob   2123   3.2  1.5 945672 250000 ?    Sl   10:55   1:30 /usr/bin/code
hakob   2301   0.0  0.2 512000  35000 pts/0 Ss+ 10:56   0:00 bash
```

- The vertical bar `|` is a pipe, which sends the output of the command on the left (`ps aux`) into the command on the right (`head`).
- `head` is a command that displays only the ↳first 10 lines of its input by default.
- If you wanted to show, say, the first 20 lines, you could use: `ps aux | head -n 20`

Columns to know

- **USER:** owner
- **PID:** process ID
- **%CPU / %MEM:** CPU and memory usage
- **VSZ / RSS:** virtual vs resident memory (RSS is real RAM used)
- **TTY:** terminal (if any)
- **STAT:** state (e.g., R running, S sleeping, D uninterruptible I/O, T stopped, Z zombie).
- **START / TIME:** start time and total CPU time
- **COMMAND:** the program and its arguments

- **-e** Select all processes.
    - Equivalent to `-A`.
    - Without it, `ps` might only show your processes or processes linked to your terminal session.
- **-o** Output format.
    - Lets you explicitly choose which columns to show and in what order.
    - You list them separated by commas, with no spaces.

In short:
- Use `aux` when you want a quick, standard, full list.
- Use `-e` + `-o` when you want control over which columns are shown.

```bash
ps -eo pid,comm

PID COMMAND
1   systemd
2   kthreadd
3   rcu_gp
4   rcu_par_gp

```

You can rename columns with `=`:

```bash
ps -o pid=P_ID,comm=COMMAND

P_ID    COMMAND
1156779 bash
1173938 ps

```

> **Important:** `-u` has two completely different meanings depending on where you put it.
>
> - **Case 1:** `ps -u <username>`
>    - **Meaning:** Select processes for this user.
>    - **Example:**
>        ```bash
>        ps -u $USER
>        ```
> - **Case 2:** `ps -u` (without a username, just the flag)
>     - **Meaning:** Show extra info in BSD-style format (adds columns like `%CPU`, `%MEM`, `START`, `TIME`, etc.).
>    - This is why `ps aux` includes `%CPU` and `%MEM`.


Combined example

```bash
ps -eo user,pid,%cpu,%mem,command

USER       PID %CPU %MEM COMMAND
root         1  0.0  0.1 /sbin/init
hakob     3201  1.2  3.4 java -jar myapp.jar
```


```bash
ps aux | grep java

# Better:
pgrep -a java     # shows matching PIDs and commands
```

Note: `grep` will also show itself; `pgrep` avoids this.

--- 

**top** — Live resource usage

`top` updates every few seconds and shows the most “hungry” processes.

Header (top lines)
- **load average** (same as uptime)
- **Tasks:** total processes, running, sleeping, etc.
- **%Cpu(s):** CPU usage split by user/system/io-wait, etc.
- **MiB Mem / Swap:** memory and swap use

Useful keys inside top
- **q**: quit
- **h**: help (shows all keys)
- **P**: sort by CPU
- **M**: sort by memory
- **1**: toggle view of each CPU core
- **k**: send a signal to a PID (same as kill) — use with care

Reading top like a pro (but in simple words)

- If load `average ≫ cores`, the system is too busy.
- If `%Cpu(s) iowait` is high, disks may be slow or blocked.
- If `Mem available` is low and `Swap used` keeps growing, you may need more RAM or to stop heavy apps.

---

**kill <PID>** — Stop or control a process (with signals)

`kill` does not always “kill.” It sends a signal. Some signals ask the app to exit cleanly.


Common signals
- **TERM (15):** polite “please exit now.” (default)
- **INT (2):** like pressing Ctrl+C
- **HUP (1):** “hang up”; many daemons reload config on HUP
- **KILL (9):** force stop immediately (last resort: no cleanup)

```
kill 12345         # send TERM (15) to PID 12345
kill -15 12345     # same as above
kill -HUP 23456    # ask process to reload (if it supports HUP)
kill -9 34567      # force stop (only if other signals failed)
```

Safe order to stop a bad process

1. Try to close the app normally (GUI or Ctrl+C).<br>
2. `kill <PID> (TERM)`.<br>
3. `kill -9 <PID>` only if it will not exit.<br>

Finding the PID first
```bash
pgrep -a myapp     # show PID(s) for myapp with command names
```

Related tools

- `pkill <name>` → send a signal by process name
- `killall <name>` → same idea (varies by distro); be careful with generic names

---

```
uname -a        # kernel + system identity
df -h           # disk usage per filesystem
free -h         # memory and swap usage (check 'available')
uptime          # time since boot + load average

ps aux          # list all processes
top             # live CPU/memory/process view
kill <PID>      # send TERM (15) by default; use -9 only last
nproc           # number of CPU cores (for load comparison)
```

---

- [HOME](./../../../README.md)
- [DevOps](./../../tutorials.md)
- [File Permissions and Ownership](./2_File_Permissions_and_Ownership.md)
- [How to Use grep in Linux: A Practical Guide](./4_How_to_Use_grep_in_Linux_A_Practical_Guide.md)