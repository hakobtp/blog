# Introduction to Git

Git is a tool that helps you track changes in your projects and work together with others. In this guide, you’ll learn how to create repositories, manage branches, handle conflicts, and collaborate effectively.

## Setting Up Git

Before using Git, you should tell it your name and email. These details will appear in your commits:

```bash
git config --global user.name "Alice Johnson"
git config --global user.email "alice.johnson@example.com"
git config --list
```

This ensures your commits are labeled correctly.

## Starting a New Project with Git

To track a project with Git, you need to initialize a repository.

1. Open a terminal and go to your project folder:
    ```bash
    cd /home/alice/projects/my-app
    ```

2. Create a Git repository:

    ```bash
    git init
    ```

Git will now keep track of changes in your folder.

3. Add your project files to the staging area (ready to be committed):
    ````bash
    git add .
    ````

---

- [Home](./../../README.md)
- [Git Tutorials](./../tutorials.md)
