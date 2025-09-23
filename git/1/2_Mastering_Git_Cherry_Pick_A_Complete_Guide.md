# Mastering Git Cherry-Pick: A Complete Guide

Git is an essential tool for developers. One powerful feature in Git is `cherry-pick`. This command allows you to take specific changes (commits) from one branch and apply them to another branch. In this post, you will learn everything about cherry-picking, from basics to advanced scenarios, including resolving conflicts, skipping commits, and editing commit messages.

## What is Git Cherry-Pick?

Imagine you have two branches: `main` and `feature-login`. You fixed a small bug on `feature-login` but do not want to merge all other changes. Cherry-pick lets you copy just that bug fix to main.

**Key Points:**
- Only selected commits are applied.
- Other commits in the source branch remain separate.
- Useful for bug fixes, hotfixes, or sharing a specific feature.

## Basic Cherry-Pick

1. Picking a Single Commit
    To apply one commit to your current branch:

    ```bash
    git cherry-pick <commit-hash>
    ```

    **Example:** You fixed a typo in `readme.md` on `feature-login` branch (commit `f3e4a2b`) and want it on main:
    ```bash
    git checkout main
    git cherry-pick f3e4a2b
    ```

    The typo fix is now applied on `main` without merging the entire `feature-login` branch.

2. Picking Multiple Commits
    You can also pick a range of commits:
    ```bash
    git cherry-pick <start-commit>^..<end-commit>
    ```

    **Example:** On `feature-ui` branch, you created three commits: add header (`a1b2c3d`), add footer (`b2c3d4e`), and add contact page (`d4e5f6g`). To move all three to main:
    ```bash
    git checkout main
    git cherry-pick a1b2c3d^..d4e5f6g
    ```

## Visualizing Cherry-Pick

Here’s a simple diagram to understand cherry-pick:

```
Feature Branch:      main branch:  
      A---B---C          X---Y---Z
           \
            D
```

- Commits `A`, `B`, `C` exist on feature branch.
- Commit `D` is the bug fix you want on main.

After cherry-picking `D`:

```
Feature Branch:      main branch:  
      A---B---C          X---Y---Z---D
           \
            D
```

Notice that only `D` is copied.

---

- [Home](./../../README.md)
- [Git Tutorials](./../tutorials.md)
- [Introduction to Git](./1_introduction_to_git.md)