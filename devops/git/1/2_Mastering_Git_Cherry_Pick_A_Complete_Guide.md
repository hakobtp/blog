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

## Handling Conflicts

Sometimes, Git cannot automatically apply the changes. This is called a conflict.

**Steps to Resolve Conflicts:**
1. Open the files with conflicts and edit them.
2. Stage the resolved files:

    ```bash
    git add <file>
    ```
3. Continue cherry-picking:

    ```bash
    git cherry-pick --continue
    ```    
4. If you want to stop the cherry-pick completely:

    ```bash
    git cherry-pick --abort
    ```

## Advanced Cherry-Pick Options

- Skip a commit:

    ```bash
    git cherry-pick --skip
    ```

- Apply without committing:

    ```bash
    git cherry-pick <commit-hash> --no-commit
    ```

- Edit the commit message:

    ```bash
    git cherry-pick <commit-hash> -e
    ```

These options give you flexibility to control how commits are applied.

## Common Use Cases

- **Bug Fixes:** Apply a fix from one branch to another without merging unrelated features.
- **Feature Sharing:** Move a single feature without merging the full branch.
- **Hotfixes:** Quickly patch a production branch with important commits.

## Example Scenario

You fixed a login bug on `bugfix-login` branch (commit `f3e4a2b`) and need it on `main`:

1. Switch to main:

    ```bash
    git checkout main
    ```

2. Cherry-pick the bug fix:

    ```bash
    git cherry-pick f3e4a2b
    ```

3. Push the updated branch:

    ```bash
    git push origin main
    ```

Now the login bug is fixed on the `main` branch without merging the full `bugfix-login` branch.

## Tips for Using Cherry-Pick

- Always check the branch you are on before cherry-picking.
- Use `git log` to find the correct commit hashes.
- Be careful with commits that depend on previous commits. Cherry-picking a dependent commit alone can cause conflicts.
- Consider creating a backup branch before cherry-picking complex changes.

## Conclusion

Git cherry-pick is a powerful tool that helps you move specific changes between branches without merging everything. By understanding how to cherry-pick, resolve conflicts, and use advanced options, you can manage your projects more efficiently and safely.

---

- [Home](./../../README.md)
- [DevOps](./../../tutorials.md)
- [Introduction to Git](./1_introduction_to_git.md)
- [Git Merge vs. Git Rebase](./3_Git_Merge_vs_Git_Rebase.md)