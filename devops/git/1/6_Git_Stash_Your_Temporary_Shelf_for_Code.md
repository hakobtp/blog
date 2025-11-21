# Git Stash: Your "Temporary Shelf" for Code

In Git, `git stash` is like putting your work on a temporary shelf. It saves all your uncommitted changes 
(both staged and unstaged) so you can get a clean working directory to do something else—without losing your work.

## Core Commands

Here are the most common commands you'll need:

**Save your changes:** This command "shelves" your changes and reverts your working directory to the last commit.

```bash
git stash
```

**Apply stashed changes later:** This applies the changes from your most recent stash back to your working directory but keeps the stash in your list.

```bash
git stash apply
```

**Apply and remove stashed changes:** This is one of the most common commands. It does the same as `apply` but also removes the stash from your list, all in one step.

```bash
git stash pop
```

**List all stashes:** You can have multiple stashes. This command shows you all the "shelves" you're using.

```bash
git stash list
```

**Clear all stashes:** This will permanently delete all of your stashes.

```bash
git stash clear
```

## Example Scenario

You’re working on a new feature, but your boss asks you to fix a critical bug. Your feature code isn't ready to be committed, but you can't fix the bug with those files changed.

1. **Stash your changes:** `git stash`
2. **Fix the bug:** Switch to the `main` branch, create a hotfix, commit your changes, etc.
3. **Return to your feature:** Switch back to your `feature` branch and run `git stash pop`.

Your feature code is back, and you can continue right where you left off.

---

- [Home](./../../README.md)
- [DevOps](./../../tutorials.md)
- [Automating your workflow with Git Hooks](./5_Automating_Your_Workflow_with_Git_Hooks.md)