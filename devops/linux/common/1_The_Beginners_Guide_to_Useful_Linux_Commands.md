# The Beginner’s Guide to Useful Linux Commands

If you’re new to Linux, using the terminal might feel strange at first. Don’t worry — with a 
few simple commands, you can move around your system, create files, and manage folders easily. Here are some of the most useful commands for beginners:


- Show current directory: **pwd** 
- List files:
    - **ls** lists the files and folders in your current location.
    - **ls -l** shows more details, like file size and date. 
    - **ls -a** also shows hidden files (their names start with a dot).
- Change directory:	**cd** `folder_name`
- Go back/up one directory:	**cd ..**
- Create a file: **touch** `file.txt`
- Create a folder: **mkdir** `myfolder`
- Delete a file: **rm** `file.txt`
- Delete a folder: 
    - **rm -r** `foldername` removes a directory and everything inside it recursively, including subfolders and files.
    - If you want extra safety, you can use: **rm -ri** `foldername` (**-i** asks for confirmation before deleting each file.)
- Move or rename a file: **mv** `file1.txt` `folder/file2.txt`
- Copy a file: **cp** `file1.txt` `file2.txt`
- View contents of a file:
    - **cat** `file.txt` quickly shows all the text in a file.
    - **less** `file.txt` lets you scroll through the file one screen at a time.
- Add text to a file:
    - **echo** `"My first DevOps lesson"` `>` `notes.txt` creates the file or replaces its contents.
    - **echo** `"My first DevOps lesson"` `>>` `notes.txt` creates the file if it doesn’t exist or appends the text if it does.    

---

- [HOME](./../../../README.md)
- [Linux](./../tutorials.md)
- [File Permissions and Ownership](./2_File_Permissions_and_Ownership.md)