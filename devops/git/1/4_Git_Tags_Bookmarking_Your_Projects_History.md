# Git Tags: Bookmarking Your Project's History

Think of Git tags like bookmarks in your project’s history. You use them to mark important versions, 
such as when you finish a new feature or release a new version (e.g., `v1.0`, `v1.1`).

## How to Create a Tag

For important releases, it's best to create an annotated tag. This type of tag stores extra information, like the author, date, and a message.

```bash
# Create an annotated (recommended) tag
git tag -a v1.0 -m "Release version 1.0"
```
You just finished version `1.0`—this tag marks that point with a clear message.

If you just want a quick, temporary tag, you can create a lightweight tag:

```bash
# Create a lightweight tag
git tag v1.0-beta
```

## How to Share Tags

By default, `git push` does not send tags to your remote repository (like GitHub). You must push them separately.

```bash
# Push just one specific tag
git push origin <tag-name>

# --- OR ---

# Push all of your local tags at once (very common)
git push origin --tags
```

This shares your tags with your team.

## Other Useful Commands


See all the bookmarks you've created.
```bash
git tag
```

If you make a mistake, you must delete the tag from two places:
```bash
# 1. Delete the tag locally
git tag -d <tag-name>

# 2. Delete the tag from the remote
git push origin --delete <tag-name>
```


## The Problem with Default Sorting

By default, **git tag** sorts alphabetically (ASCII order). This causes a common problem:

```
# A typical list of tags, unsorted
v1.0
v1.10
v1.2
v2.0
v2.1
```

Notice that `v1.10` comes before `v1.2` because the text `"10"` comes before `"2"`. This is not what we want.

To sort this list correctly, you use the `--sort=version:refname` flag. `refname` just means "the tag's name," and `version`: tells Git to treat that name like a version number.

```bash
git tag --sort=version:refname

# The list is now correctly sorted
v1.0
v1.2
v1.10
v2.0
v2.1
```

If you want to see the newest version first, just add a minus sign (`-`) to the sort key.

```bash
git tag --sort=-version:refname

# The list is now in descending (newest) order
v2.1
v2.0
v1.10
v1.2
v1.0
```
## GPG-signed tag

- `-s (--sign)`**:** Creates a GPG-signed tag. Think of this as a digital, unforgeable signature on your "bookmark." 
    It proves that you (and only you) created this tag, and it hasn't been tampered with. This is crucial for security and verifying official releases.
- `-u <key-id> (--local-user)`**:** Tells Git which specific GPG key to use for signing. You only need this if you have multiple GPG keys 
    (e.g., one for work, one for a personal project) and don't want to use your default key.    

Let's imagine you have two different GPG keys set up on your machine.

**Step 1: Find Your Key ID**
First, you need to know the "`Key ID`" of the key you want to use. You can list your secret keys with this command:

```bash
gpg --list-secret-keys --keyid-format=long
```

The output will look something like this:
```
/home/username/.gnupg/pubring.kbx
----------------------------------
sec   rsa4096/AABBCC112233 2024-01-10 [SC]
      FINGERPRINT = ...
uid   [ultimate] Your Name (Personal Key) <personal@example.com>

sec   rsa4096/998877FFEEAA 2025-03-15 [SC]
      FINGERPRINT = ...
uid   [ultimate] Your Name (Work Key) <name@company.com>
```

In this example, your Key IDs are `AABBCC112233` (Personal) and `998877FFEEAA` (Work).

**Step 2: Create the Signed Tag**
Let's say your default key is your personal one, but you are tagging a work release and must use your work key.

This is where you use `-u` to select the specific key.

```bash
# -s tells Git to sign the tag
# -u 998877FFEEAA tells Git to use your "Work Key"
# -m provides the tag message
git tag -s -u 998877FFEEAA v2.5.1 -m "Version 2.5.1 release for production"
```

When you run this, Git will prompt you for the passphrase for your `998877FFEEAA` GPG key to complete the signature.

> **Note:** If you only use `-s` without `-u`, Git will simply use your default key (the one you specified in your `.gitconfig` file with `user.signingkey`).

```bash
# This will sign the tag using your default GPG key
git tag -s v1.0-beta -m "Beta release"
```

**Step 3: Verify the Signature**

Anyone (including you) can now check the tag's signature to prove it's authentic:

```bash
git tag -v v2.5.1
```

If the signature is valid, you'll see a confirmation message showing it was signed by you.

### Why You Need Signed Tags

Git is built on trust. By default, nothing stops someone from impersonating you. 

Imagine a bad actor clones your project. They could easily run these commands:

```bash
# 1. They pretend to be you
git config user.name "Your Name"
git config user.email "your@email.com"

# 2. They create a fake, malicious version and tag it
git tag v1.0-malicious

# 3. They push the tag to the public repository
git push origin v1.0-malicious
```

How does anyone know you didn't create that tag? They can't. This is a huge security risk.

**The Solution: A Digital Signature**

This is where the `-s` flag saves the day.

When you create a signed tag, you use a private `GPG key`—a secret password that only you possess. This creates a unique, unforgeable digital signature.
Now, others can use your `public key` to verify the tag.

- **Authenticity:** If it verifies, they know for certain it was created by you.
- **Integrity:** They also know the code hasn't been tampered with since you signed it.

If a bad actor tries to fake a tag, it will fail this verification.

#### A Simple Analogy

This is the easiest way to think about it:
- A regular tag (`git tag v1.0`) is like a sticky note. Anyone can write "Version 1.0" on it and stick it on.
- A signed tag (`git tag -s v1.0`) is like a legal document that you have personally signed and had notarized. It is provably from you and only you.

### So, When Should I Use It?

You should use signed tags whenever it is critical for people to be 100% sure your project release is authentic and safe.

This is essential for:
- Major open-source projects
- Any company with strict security policies

For your day-to-day work, a simple annotated tag (`-a`) is fine. 
But when publishing an official, public release, signing it (`-s`) is the professional and secure way to go.

### These flags are used when you are making a new tag.

|Flag|	Description|
|:---|:------------|
|`-a`, `--annotate`|Creates an **annotated tag**. This is the recommended flag for releases as it stores extra data like the tagger, date, and a message.|
|`-m <message>`|Provides the **tag message** for an annotated tag. If you use `-a` without `-m`, Git will open your text editor to ask for a message|
|`-s`, `--sign`|Creates a **GPG-signed tag**. This is an annotated tag that also includes a digital signature to prove who created it.|
|`-f`, `--force`|**Forces** the tag to be created or updated. Use this if you need to replace a tag that already exists (use with caution).|
|`-F <file>`|Takes the tag message from a specific file instead of you typing it with `-m`.|
|`-u <key-id>`|Used with `-s` to specify which GPG key to use for signing.|

### These flags control how you view the tags you already have.

|Flag|	Description|
|:---|:------------|
|`-l`, `--list`|Lists all tags. This is the default action if you just type `git tag`. You can also use it with a pattern (e.g., `git tag -l "v1.*"` to find all tags starting with "`v1`.").|
|`-n<num>`|When listing, this shows the first `<num>` lines of the tag message. For example, `git tag -n1` will list all tags and the first line of their annotation.|
|`--sort=<key>`|Sorts the list of tags. A very common use is `--sort=version:refname` (or `-v:refname`), which sorts version numbers correctly (so `v1.10` comes after `v1.2`).|
|`--contains <commit>`|Lists only the tags that contain a specific commit. This is useful for seeing which release a certain feature or bugfix is in.|
|`--points-at <object>`|Lists only the tags that point directly at a specific commit or object|

### These are for managing existing tags.

|Flag|	Description|
|:---|:------------|
|`-d`, `--delete`|Deletes a tag from your local repository. (Remember, you also need to use `git push origin --delete <tag-name>` to remove it from the remote)|
|`-v`, `--verify`|Verifies the GPG signature of a signed tag. This checks if the tag is authentic and has not been tampered with.|

---

- [Home](./../../README.md)
- [DevOps](./../../tutorials.md)
- [Git Merge vs. Git Rebase](./3_Git_Merge_vs_Git_Rebase.md)
- [Automating your workflow with Git Hooks](./5_Automating_Your_Workflow_with_Git_Hooks.md)