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

**Example:** You’ve finished the login form in `feature-login`. Merging it into `main` makes it live for the whole project.

Think of your project's history as a story.

- `git merge` adds a new chapter that says, "At this point, the stories from the `main` branch and the 
    `feature-login` branch were combined." It preserves the exact history of both branches and ties them 
    together with a special merge commit. This creates a graph-like history.

- `git rebase` takes the chapters you wrote in your `feature-login` branch, goes back to where it split from `main`, 
    fast-forwards `main` to its latest version, and then rewrites your chapters one by one on top of it. This creates a clean, 
    linear history, making it look as if you did all your work sequentially.


| Feature | `git merge` | `git rebase` |
| :--- | :--- | :--- |
| **Commit History** | Preserves the full, branching history and adds a "merge commit". | Creates a clean, perfectly linear history by rewriting commits. |
| **Branch Impact**| Non-destructive. The original branch histories are left unchanged. | Potentially destructive. It rewrites the history of the feature branch. |
| **Primary Use Case**| Combining a feature branch into a shared public branch (e.g., `main`). | Cleaning up a local, private feature branch *before* merging. |

## Removing Branches

After finishing a feature, you can delete the branch to keep your repository clean.

```bash
git branch -d feature-login    # Delete branch locally
git push origin :feature-login # Delete branch on the remote server
```

## Solving Merge Conflicts

Sometimes, Git cannot combine branches automatically. You need to fix conflicts manually.

Check differences:
```bash
git diff
git diff main feature-login
```

Then edit the files, and save your changes:
```bash
git add index.html
git commit
```

**Example:** If you and your teammate both change the same paragraph in index.html, Git will ask you to choose which change to keep.

## Working with Remote Repositories

A remote repository is like a copy of your project stored on GitHub or GitLab. You can link your local repository to it:

```bash
git remote add origin https://github.com/yourname/project.git
git remote set-url origin https://github.com/yourname/new-project.git
git remote rename origin main-remote
```

Send your work to the remote repository:

```bash
git push -u origin feature-login  # Upload branch and set upstream
git pull origin feature-login     # Download changes from remote
```


Update your local copy without merging:

```bash
git fetch
```

`git fetch` downloads changes from the remote repository (like GitHub) without changing your current files. 
It just updates your local copy of the remote branches. Think of it like checking if your teammate has done something new, but not touching your own work yet. After this command, Git knows what new commits exist on the remote repository, but your current branch doesn’t change.

**Example:**

Imagine you and a friend are working on a website:
1. You are on branch main.
2. Your friend adds a new page called about.html on the remote repository.
3. You run:
    ```bash
    git fetch
    ```

- Git now knows that about.html exists on the remote.
- Your local main branch hasn’t changed yet.
- You can look at the changes, or merge them later when you’re ready.

How it’s different from `git pull`

- `git pull` = `git fetch` + `git merge` Pull downloads the changes and automatically merges them into your current branch.
- `git fetch` only downloads the changes. You choose when and how to merge them.

```bash
git fetch          # Just update your knowledge of the remote
git merge origin/main  # Merge the remote main into your current branch
```

Quick analogy:

- `git fetch` = Checking the mailbox
- `git pull` = Checking the mailbox and taking the letters inside

## Undoing Changes

You can undo changes in a file or revert commits if something goes wrong.

```bash
git reset app.js             # Unstage changes in a file
git reset --soft HEAD~1      # Undo last commit, keep changes staged
git reset --mixed HEAD~1     # Undo last commit, keep changes in files
git reset --hard HEAD~1      # Undo last commit and delete changes
```

Revert a commit safely:

```bash
git revert 1a2b3c4           # Create a new commit that undoes '1a2b3c4'
```

**Example:** If you accidentally added a wrong feature, git revert lets you undo it without rewriting history.

## Saving Work Temporarily: Stash

Sometimes you need to switch branches but aren’t ready to commit your work. Stash it!

```bash
git stash       # Save your changes
git stash pop   # Reapply and remove stash
git stash apply # Reapply without removing stash
git stash clear # Remove all stashes
```

**Example:** You’re halfway through designing a homepage but need to fix a bug on another branch. Stashing saves your unfinished work safely.


## Pushing Your Branch

Finally, share your branch with others:

```bash
git push origin feature-login
```

This sends your branch to the remote repository so your team can see and work with it.

---

- [Home](./../../README.md)
- [DevOps](./../../tutorials.md)
- [Mastering Git Cherry-Pick: A Complete Guide](./2_Mastering_Git_Cherry_Pick_A_Complete_Guide.md)
