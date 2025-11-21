# Git Merge vs. Git Rebase

Have you ever opened your Git history and felt lost in a maze of branches and commits? 
It can look like a plate of tangled spaghetti! When you’re trying to find where a bug started, a messy history makes your job much harder.

In Git, you often need to combine work from different branches. The two main ways to do this are `merge` and `rebase`.

Both can bring changes from one branch into another, but they do it in very different ways—and that difference changes how your project's story is told.

Let’s explore both.

## Git Merge

`git merge` joins two branches together. It takes all the changes from another branch and combines them into 
the one you’re on. To tie everything together, it creates a special **merge commit**.

```bash
# While on your main branch
git merge <feature-branch-name>
```

Merging keeps the entire history of both branches, exactly as it happened. Git simply connects their timelines with this new merge commit, which shows where the two histories came together.

Example: You are on your `main` branch and want to bring in work from your `user-auth` branch.

You run:

```bash
git checkout main
git merge user-auth
```

Now `main` includes everything from `user-auth`, and Git adds a merge commit to record this event.

```
Before the merge: 'main' is at C, 'user-auth' is at E.

      C  <--(main)
     /
A---B
     \
      D---E  <--(user-auth)


After the merge: A new commit 'M' is created. 'M' has TWO parents: C and E. 'main' now points to M.

      C-------M  <--(main)
     /       /
A---B-------/
     \     /
      D---E  <--(user-auth)
```      

**Important:** Merge preserves the exact history and adds a new commit to show where the branches were joined.

**Advantages:**
- **Preserves History:** Keeps the complete, original commit history. Nothing is rewritten.
- **Traceable:** It's easy to see when and where branches were joined.
- **Safe:** This is the safest option for shared, public branches (like `main` or `develop`) where many people are collaborating.

**Disadvantages:**
- **"Noisy" History:** Can make the commit log (history) longer and harder to follow.
- **Clutter:** Creates many merge commits, which can clutter the log if used for every small update.

## Git Rebase

`git rebase` also combines changes, but in a completely different way. Instead of joining two histories, it rewrites your branch's history.

It temporarily "lifts" your new commits, fetches the latest changes from the other branch, and then "re-plays" your commits one-by-one on top of those new changes.

```bash
# While on your feature branch
git rebase <base-branch-name>
```

Rebasing makes your history look like you started your work after the latest changes were added, resulting in a clean, straight-line (linear) history.

**Example:** You’re working on a branch called `payment-flow`. While you were coding, your teammates added new commits (`C` and `D`) to `main`. 
Your `payment-flow` branch is now out of date.

To update your branch, you run:

```bash
git checkout payment-flow
git rebase main
```

```
Before rebase: 'payment-flow' started after B. 'main' has moved on.

      E---F  <--(payment-flow)
     /
A---B---C---D  <--(main)


After rebase: Your commits (E, F) are re-applied as new commits (E', F')
on top of 'main'. Your 'payment-flow' branch pointer is moved.

                  E'---F'  <--(payment-flow)
                 /
A---B---C---D          <--(main)
```

**Important:** Rebase rewrites your branch history to appear after the latest commits. The original commits (`E`, `F`) 
are gone, and new ones (`E'`, `F'`) are created. The `main` branch itself is not changed.

**Advantages:**
- **Clean History:** Creates a perfectly straight, linear history that is easy to read.
- **No Clutter:** Avoids the extra merge commits.
- **Good for Updates:** Great for updating your personal feature branch with the latest changes from `main`.

**Disadvantages:**
- **Rewrites History:** This is dangerous if other people are using the same branch. (See the Golden Rule below!)
- **Loses Context:** You lose the original context of when work actually started.
- **Complex Conflicts:** If you have conflicts, you may have to solve them for each commit as it's re-played, which can be tricky.

## Merge vs. Rebase: At a Glance

|Aspect |Git Merge |Git Rebase|
|:------|:---------|:---------|
|`History`|Preserves original history. Adds a merge commit.|Rewrites history. Creates new commits for a linear log.|
|`Commit Log`|Can be "noisy" or "spaghetti-like" with many joins.|Clean, straight-line, and easy to read.|
|`Conflicts`|Solved once at the moment of the merge.|Can be solved multiple times, once for each re-played commit.|
|`Safety`|Safe for shared/public branches.|Risky for shared branches. Safe for your local-only branches.|
|`Best For`|Integrating a feature into `main`; shared branches.|Cleaning up your own feature branch before merging.|

## When to Use Each
This isn't an "either/or" choice. The best practice is to use both tools for what they do best.

- Use `git rebase` when:
    - You are working on your own local feature branch.
    - You want to "catch up" your branch with the latest changes from `main`.
    - You want to clean up your commits before you share them with the team.
- Use `git merge` when:
    - You are integrating a completed feature branch into a shared branch like `main` or `develop`.
    - You want to keep a full, explicit history of when and where branches were joined.    

## The Best of Both: A Professional Workflow

This leads to the "Rebase, then Merge" workflow, which is used by most professional teams. It gives you a clean history and a safe, traceable record.

**Imagine:**
- You’re developing a feature on a branch called `dark-mode`.
- Meanwhile, new commits are added to `main`.

Your branch is now behind `main`. Here's the clean way to handle it:

**Step 1: Rebase Your Feature Branch (Locally)**

First, update your local `main` branch, then "re-play" your `dark-mode` changes on top of it.

```bash
# 1. Get the latest changes from the remote
git checkout main
git pull

# 2. Go back to your feature branch
git checkout dark-mode

# 3. Rebase your changes on top of the new main
git rebase main
```

Now your `dark-mode` branch includes all the latest `main` commits, and your new commits are neatly placed at the end. It's as if you just started your feature today!

**If conflicts appear:** Git will stop and let you fix them.
1. Fix the files listed.
2. Run `git add .` to mark them as resolved.
3. Run `git rebase --continue` to keep going.

**Step 2: Merge Your Clean Branch into** `main`

Once your branch is rebased and working, you're ready to merge it into `main`.

```bash
# 1. Switch to the main branch
git checkout main

# 2. Merge your feature branch
git merge dark-mode
```

Because you rebased first, `dark-mode` is now a direct descendant of `main`. This merge will be a "**fast-forward**"—Git simply moves the `main` pointer forward to include your new commits. No "merge commit" is needed, and the history stays perfectly linear.

**Optional (but recommended): Keep a Record with** `--no-ff`

A fast-forward merge is clean, but it erases the context of which commits belonged to your feature. It can look like all that work was done directly on `main`.

To get the best of both worlds, use the `--no-ff` (no fast-forward) flag.

```bash
git checkout main
git merge --no-ff dark-mode
```

This forces Git to create a merge commit, even though it could fast-forward. This gives you a clean, linear history plus a single merge commit that groups your feature's work together.

It’s like putting a label on your history that says, "The 'dark-mode' feature was completed and added right here."

## The Golden Rule of Rebase

> **Never rebase a branch that other people are also using.**

Rebasing rewrites history. If you rebase a shared branch, you are creating a new, different history. 
When your teammates `pull` your rebased branch, Git will see their local history and your new history as 
two different things, leading to massive confusion and "spaghetti" problems.

> **Only rebase your own local branches that you haven't shared yet.**


## Your New Workflow: A 3-Step Summary

|Step|Command|What It Does|
|:---:|:------|:-----------|
|1️⃣|`git rebase main`|(On your feature branch) Updates your branch with the newest changes from `main`.|
|2️⃣|`git checkout main`|Switch to the `main` branch.|
|3️⃣|`git merge --no-ff <feature>`|Merge your feature, keeping its commits grouped together.|

---

- [Home](./../../README.md)
- [DevOps](./../../tutorials.md)
- [Mastering Git Cherry-Pick: A Complete Guide](./2_Mastering_Git_Cherry_Pick_A_Complete_Guide.md)
- [Git Tags: Bookmarking Your Project's History](./4_Git_Tags_Bookmarking_Your_Projects_History.md)