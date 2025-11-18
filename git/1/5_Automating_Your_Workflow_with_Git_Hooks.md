# Automating your workflow with Git Hooks

Git hooks are like automatic helpers for your repository. They are scripts that Git runs automatically when specific 
actions happen (like committing or switching branches), helping you save time and enforce quality.

## Common Hooks

- **pre-commit:** Runs before you even type a commit message. This is perfect for running 
    linters or automated tests to catch errors before they get into your codebase.
- **post-checkout:** Runs after you successfully switch branches. This is useful for tasks 
    like automatically installing new dependencies if a package.json or requirements.txt file changed.

## How to Set Up a Hook

Hooks live inside your project's `.git/hooks` folder. By default, this folder is filled with sample files.

1. **Find the Sample:** Look for the sample file you want to use, like `pre-commit.sample`.
2. **Activate the Hook:** To activate it, just rename the file by removing the `.sample` extension:
    ```bash
    mv .git/hooks/pre-commit.sample .git/hooks/pre-commit
    ```
3. **Make it Executable:** Git requires the script to be executable. Use chmod to set the permission:    
    ```bash
    chmod +x .git/hooks/pre-commit
    ```
4. **Write Your Script:** Now you can open the `pre-commit` file and write your logic. This is typically a shell script, but you can use other languages like Python or Ruby. 

**Example1:** A hook script to prevent direct commits to 'main' or 'master'

```bash
#!/bin/sh
#
# A hook script to prevent direct commits to 'main' or 'master'.

current_branch=$(git rev-parse --abbrev-ref HEAD)

if [ "$current_branch" = "main" ] || [ "$current_branch" = "master" ]
then
  echo "---------------------------------------------------------------"
  echo "Error: Direct commits to the '$current_branch' branch are forbidden!"
  echo "Please create a feature branch (e.g., 'git checkout -b my-feature')"
  echo "and commit your changes there."
  echo "---------------------------------------------------------------"
  exit 1
else
  exit 0
fi
```
- **How It Works:**
    - `#!/bin/sh`**:** This is called a "shebang." It tells your system to run this file using the standard shell.
    - `current_branch=$(git rev-parse --abbrev-ref HEAD)`**:** This command asks Git, "What's the short name of the branch I'm currently on?"
        and stores the answer (e.g., `main` or `my-feature`) in the `current_branch` variable.
    - `if [ "$current_branch" = "main" ] || [ "$current_branch" = "master" ]`**:** This is a simple logic check. 
        It says, "If the current branch variable is 'main' or 'master'..."    
    - `echo "..."`**:** If the `if` statement is `true`, this prints your custom error message to the terminal. I've added the `---` lines to make it stand out.
    - `exit 1`**:** This is the most important part. Exiting with a `1 (or any non-zero number) tells Git "Stop! The check failed." Git will then abort the commit process            
    - `exit 0`**:** If the if statement is false (meaning you're not on main or master), this exits with a `0`. This tells Git, "All good! The check passed," and the commit is allowed to continue.

**Example1:** A hook to check if the commit message starts with a ticket ID (e.g., JIRA-123: Message)

1. Create a new file in your hooks folder: `.git/hooks/commit-msg`.
- Make it executable: `chmod +x .git/hooks/commit-msg`.

Paste the following code into that file:

```bash
#!/bin/sh
#
# A hook to check if the commit message starts with a ticket ID
# (e.g., JIRA-123: Message)

# Get the commit message from the file passed as an argument
commit_msg_file=$1
first_line=$(head -n1 "$commit_msg_file")

# Define the regular expression for the pattern (e.g., JIRA-123: Message)
# This checks for one or more uppercase letters, a hyphen, one or more numbers,
# a colon, and a space.
regex="^[A-Z]+-[0-9]+: .+"

# Check if the first line matches the regex
if ! echo "$first_line" | grep -qE "$regex"; then
  echo "---------------------------------------------------------------"
  echo "Error: Invalid commit message format!"
  echo "Your message must start with a ticket ID, like:"
  echo "  'JIRA-123: Fix login bug'"
  echo "---------------------------------------------------------------"
  exit 1
fi

exit 0
```

- **How It Works:**
    - `commit_msg_file=$1`**:** Unlike `pre-commit`, the `commit-msg` hook is given an argument by Git: the path to a temporary file that contains your commit message. We store this path.
    - `first_line=$(head -n1 "$commit_msg_file")`**:** We read just the first line from that file, as that's all we need to check.
    - `regex="^[A-Z]+-[0-9]+: .+"`**:** This is a regular expression.
        - `^` means "starts at the beginning of the line."
        - `[A-Z]+` means "one or more uppercase letters."
        - `-[0-9]+` means "a hyphen, followed by one or more numbers."
        - `:` means "a colon and a space."
        - `.+` means "any character, one or more times" (your actual message).
    - `if ! echo "$first_line" | grep -qE "$regex"`**:** This is the test.    
        - `echo "$first_line"` prints your message.
        - `|` (pipe) sends that message to the grep command.
        - `grep -qE "$regex"` quietly (`-q`) checks if the message matches the extended regex (`-E`).
        - The `!` inverts the logic, so it means "If the message does not match..."
    - `exit 1`**:** If the match fails, it prints the error and exits with `1` to abort the commit.
    - `exit 0` **:** If the message does match, the script exits cleanly, and your commit is saved.    

- **How to Test It**
    - Make a change and try to commit with a bad message: `git commit -m "Fix bug"` (The hook will run, show your error, and stop the commit.)
    - Now, try to commit with a good message: `git commit -m "TASK-456: Fix bug"` (The hook will run, pass silently, and the commit will be successful.)


Think of hooks as a smart assistant that reminds you to double-check your work (like running a spell-check) before you're allowed to send the email.

---

- [Home](./../../README.md)
- [Git Tutorials](./../tutorials.md)
- [Git Tags: Bookmarking Your Project's History](./4_Git_Tags_Bookmarking_Your_Projects_History.md)
- [Git Stash: Your "Temporary Shelf" for Code](./6_Git_Stash_Your_Temporary_Shelf_for_Code.md)