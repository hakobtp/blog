# File Permissions and Ownership

Linux controls who can access and modify files using a **permission system**.
Knowing how permissions work helps you protect your files and avoid unwanted changes.

## The Permission Model in Linux

Every file and folder in Linux has three types of owners:

| Owner Type | Who It Is                                                     |
| ---------- | ------------------------------------------------------------- |
| **User**   | The person who owns the file (usually the one who created it) |
| **Group**  | A set of users with shared access rights                      |
| **Others** | Anyone else on the system                                     |

Each of these can have three permissions:

| Permission  | Symbol | What It Does                                         |
| ----------- | ------ | ---------------------------------------------------- |
| **Read**    | `r`    | View the file’s contents (or list files in a folder) |
| **Write**   | `w`    | Modify the file (or add/delete files in a folder)    |
| **Execute** | `x`    | Run the file as a program or script                  |

Diagram: How Linux Permissions Work

```sql
- r w x   r - x   r - -
  user    group   others
```

- First character (**-**) is file type 
    - **-** regular file
    - **d** directory
    - **l** symbolic link, and other special file types like 
    - **b** block device file (e.g., hard drive, USB)
    - **c** character device file (e.g., keyboard, serial port)
    - **p** named pipe (FIFO)
    - **s** socket file

- Next 3 → user permissions
- Next 3 → group permissions
- Last 3 → others’ permissions

Use the `ls -l` command  for checking File permissions

```bash
ls -l

-rwxr-xr-- 1 john devops 1024 Aug 14 script.sh
```

Breakdown:
- `-rwxr-xr--`  permission string
- `john`  user (owner)
- `devops`  group
- `script.sh`  file name


How to Read `-rwxr-xr--`

| Position | Owner Type | Permissions | Meaning              |
| -------- | ---------- | ----------- | -------------------- |
| 1–3      | User       | rwx         | Read, write, execute |
| 4–6      | Group      | r-x         | Read, execute        |
| 7–9      | Others     | r--         | Read only            |

## Changing File Permissions

The command is:

```bash
chmod [permissions] filename
```

Example:

```bash
chmod 755 script.sh
```

This means:

| Number | Permissions | Meaning              |
| ------ | ----------- | -------------------- |
| 7      | rwx         | read, write, execute |
| 5      | r-x         | read, execute        |
| 5      | r-x         | read, execute        |


How numbers map to permissions:

| Number | Binary | Permissions |
| ------ | ------ | ----------- |
| 7      | 111    | rwx         |
| 6      | 110    | rw-         |
| 5      | 101    | r-x         |
| 4      | 100    | r--         |
| 0      | 000    | ---         |



---

- [HOME](./../../../README.md)
- [Linux](./../tutorials.md)
- [The Beginner’s Guide to Useful Linux Commands](./1_The_Beginners_Guide_to_Useful_Linux_Commands.md)