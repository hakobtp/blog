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
$ uname -a
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

- [HOME](./../../../README.md)
- [Linux](./../tutorials.md)
- [File Permissions and Ownership](./2_File_Permissions_and_Ownership.md)