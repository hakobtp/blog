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

---

- [Home](./../../README.md)
- [Git Tutorials](./../tutorials.md)
- [Introduction to Git](./1_introduction_to_git.md)