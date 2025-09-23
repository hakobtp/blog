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

4. Save your first commit:

    ```bash
    git commit -m "First commit: project setup"
    ```

## Cloning an Existing Project

Sometimes, you want to copy a project that already exists online. This is called cloning.

Get the project’s URL (HTTPS or SSH), for example:

- **HTTPS:** https://github.com/alice/my-app.git
- **SSH:** git@github.com:alice/my-app.git

Clone it like this:

```bash
git clone <repository-url>
```

This makes a local copy on your computer.

## Working with Remote Repositories

If you create a new repository on a platform like GitHub or GitLab, follow these steps to link it with your local project:

```bash
git remote add origin <repository-url>
git branch -M main
git push -u origin main
```

Now your local project is connected to the online repository.

## Using Branches

Branches let you work on new features without affecting the main project. Create a new branch:

```bash
git checkout -b new-feature
```

Make changes, add them, commit, and push:

```bash
git add .
git commit -m "Add new login page"
git push origin new-feature
```

## Tracking Changes

Git keeps track of file changes in two main steps: staging and committing.

- Stage files (prepare them to be committed):
```bash
git add index.html         # Stage a single file
git add .                  # Stage all files
```

- Commit files (save changes to the repository):
```bash
git commit -m "Add homepage design"
git commit -am "Update styles"   # Add and commit tracked files in one step
git commit --amend              # Change the last commit message
```

## Checking Status and History

To see what has changed or needs committing:
```bash
git status
```

To see previous commits:

```bash
git log            # Full history
git log --oneline  # Short summary
```

## Example Scenario

Imagine you are working on a website. You start with `index.html` and `style.css`. You add these files to Git (`git add .`) and commit (`git commit -m "Initial website setup"`). Later, you create a branch contact-page, add a new `contact.html`, commit, and push. This way, your main site stays safe while you work on new pages.

## Creating and Switching Branches

You can create a new branch to work on a feature or fix a bug. Then, you can switch between branches whenever you need.

```bash
git branch feature-login     # Make a new branch called 'feature-login'
git checkout feature-login   # Move to the 'feature-login' branch
git checkout -b feature-ui   # Make a new branch and switch to it at the same time
```

**Example:** Imagine your project is a website. You want to add a login form without breaking the main website. You create a `feature-login` branch and work there. The `main` branch remains safe while you test your changes.

## Combining Changes: Merge or Rebase

When your work is ready, you can bring your branch changes into another branch.

```bash
git merge feature-login  # Combine changes from 'feature-login' into your current branch
git rebase main          # Reapply your branch changes on top of 'main'
```

**Example:** You’ve finished the login form in `feature-login`. Merging it into `main` makes it live for the whole project. Rebasing is useful if you want a cleaner history without extra merge commits.

---

- [Home](./../../README.md)
- [Git Tutorials](./../tutorials.md)
- [Mastering Git Cherry-Pick: A Complete Guide](./2_Mastering_Git_Cherry_Pick_A_Complete_Guide.md)
