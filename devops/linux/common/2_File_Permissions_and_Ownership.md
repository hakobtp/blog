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

---

- [HOME](./../../../README.md)
- [Linux](./../tutorials.md)
- [The Beginner’s Guide to Useful Linux Commands](./1_The_Beginners_Guide_to_Useful_Linux_Commands.md)