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

- [HOME](./../../../README.md)
- [Linux](./../tutorials.md)
- [File Permissions and Ownership](./2_File_Permissions_and_Ownership.md)