## Setting Up a Python Project


```
Author: Ter-Petrosyan Hakob
```

---

When starting a new Python project, managing dependencies and environment-specific settings is crucial to avoid 
**“It works on my machine”** problems. In this post, we’ll cover:

- **Virtual environments (venv):** what they are, why they matter, and how to set them up.
- **.env files:** storing sensitive or environment-specific variables safely.
- **requirements.txt:** pinning Python packages for reproducible installs.
- **environment.yml:** using Conda to manage both Python and non-Python dependencies, including best practices and key differences from `requirements.txt`.

By the end, you’ll understand not only how to create each file but also when and why to use each tool in your workflow.

## Virtual Environments (venv)

A virtual environment (commonly abbreviated **“venv”**) is a self-contained directory that holds a specific Python interpreter and a set of packages isolated from your system’s global Python installation. In essence, it allows every project to maintain its own dependencies without interfering with other projects.

**Why isolation matters:**
- You might need Django 3.2 for one project and Django 4.2 for another. Without isolation, upgrading Django system-wide could break older projects.
- Some libraries require conflicting versions of the same package. Virtual environments prevent clashes.

### How venv Works under the Hood

When you run: 

```bash
python3 -m venv venv
```
Python creates a folder named `venv/` containing:
- A Python interpreter (often a copy of or symlink to your system Python).
- A pip binary.
- Subdirectories for installed packages (`lib/pythonX.Y/site-packages` on Linux/macOS or `Lib\site-packages` on Windows).

Once you **activate** this environment, any `pip install` commands will install into `venv/lib/...` (rather than the global site-packages). Deactivating returns your shell to the system interpreter.

> If you have multiple Pythons (e.g., Python 3.9 and 3.11), specify exactly: ```python3.11 -m venv venv```

**Activate the venv:**

```bash
# macOS/Linux (bash, zsh):

source venv/bin/activate

```

```powershell
# Windows PowerShell:

.\venv\Scripts\Activate.ps1

```

After activation, your prompt usually shows (`venv`) at the beginning, indicating you’re inside that environment

```bash
(venv) ~
```

**Deactivate when you’re done:**

```bash
deactivate

```

This returns you to the system Python.

### Best Practices and Tips

- **Name conventions:** It’s common to name the folder `venv/`, `.venv/`, or `env/`. Prefixing with a dot (`.venv/`) hides it from most directory listings.
- **Add venv to .gitignore:** You do not want to commit the entire virtual environment. Instead, share only `requirements.txt` (or `environment.yml`). 
- **Pin Python version:** If you rely on language features introduced in Python 3.10 (for example, structural pattern matching), 
    consider documenting “Python 3.10+” in your README so collaborators know which interpreter to use.
- **Activate automatically (optional):** Tools like direnv can auto-load environment variables and activate your 
    venv when you cd into the project folder. This can save you from typing source `venv/bin/activate` every time.    


---

## Explore More

- [Home](./../../README.md)
- [Python Tutorials](./../tutorials.md)