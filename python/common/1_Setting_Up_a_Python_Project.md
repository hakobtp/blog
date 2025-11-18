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

## .env Files

A `.env` file holds environment variables—`key/value` pairs you don’t want to hard-code in your source. Typical uses include:

- API keys or tokens (e.g., `STRIPE_SECRET_KEY=sk_test_…`).
- Database URLs or credentials (`DATABASE_URL=postgres://user:pass@localhost:5432/mydb`).
- Debug flags or environment settings (`DEBUG=True, FLASK_ENV=development`).

By reading from a `.env` file at runtime, your code can:
- Distinguish between development, staging, and production environments without changing code,
- Avoid accidentally committing secrets to version control,
- Simplify configuration management (one place to set values).

**Creating and Loading .env**

Create the .env file in your project root (e.g., next to `main.py`, `app.py`, or `manage.py`):

```bash
touch .env
```

Add key/value pairs (one per line). Example:

```bash
SECRET_KEY=MySuperSecretKeyXYZ
DATABASE_URL=postgres://alice:secret@localhost:5432/myapp
DEBUG=True

```

- **Spacing matters:** Do not surround the equals sign with spaces.
- **Comments begin with #**, but some parsers ignore lines starting with **#**. Always confirm your parser’s rules.

Install and use `python-dotenv` (optional, but recommended):

```bash
pip install python-dotenv
```

Then, in your code (`main.py` or `app.py`), add:

```python
from dotenv import load_dotenv
import os

# This reads the .env file and sets the environment variables accordingly.
load_dotenv()

SECRET_KEY = os.getenv("SECRET_KEY")
DATABASE_URL = os.getenv("DATABASE_URL")
DEBUG = os.getenv("DEBUG", "False").lower() in ("true", "1", "yes")
```

Many frameworks (e.g., Flask, Django with django-environ) provide built-in or community-supported ways to load `.env` automatically.

Add `.env` to `.gitignore`:

```bash
echo ".env" >> .gitignore
```

This ensures you never push secrets to the repository. If collaborators need default or sample values, include a file called `.env.example` (without real passwords) showing which keys are expected.

### Best Practices

- Never commit secrets. Always keep the real `.env` out of version control.
- Use a `.env.example` or `.env.sample`. Provide placeholder values so new team members know which variables they must define. For example:

    ```bash
    SECRET_KEY=your-secret-key-here
    DATABASE_URL=postgres://user:password@localhost:5432/dbname
    DEBUG=False
    ```
- **Differentiate environments.** You might have `.env.development`, `.env.staging`, and `.env.production`. 
    Your deployment pipeline or CI/CD can choose which one to load.
- **Avoid storing sensitive files in Docker images.** Instead, pass secrets as Docker environment variables or use a secrets manager 
    (e.g., AWS Secrets Manager, HashiCorp Vault).

```
my_app/
├── .env              ← general defaults (optional)
├── .env.local        ← local overrides (not committed)
├── .gitignore
└── app.py
```

```python
from dotenv import load_dotenv
import os

# 1) First load the default .env (if you have one)
load_dotenv(dotenv_path=".env", override=False)

# 2) Then load .env.local, overriding any values from .env
load_dotenv(dotenv_path=".env.local", override=True)

# Now os.getenv() will reflect values from .env.local for any keys defined there.
SECRET_KEY = os.getenv("SECRET_KEY")
DEBUG = os.getenv("DEBUG", "False").lower() in ("true", "1", "yes")

print("Secret Key:", SECRET_KEY)
print("Debug mode:", DEBUG)
```

If you don’t want to hardcode the filename, you can use `find_dotenv()` to discover the nearest dotenv file. 
By default, `find_dotenv()` searches for a file named `.env`. To find `.env.local`, call it with the filename:

```python
from dotenv import load_dotenv, find_dotenv
import os


# Load default .env if present
default_path = find_dotenv(".env", raise_error_if_not_found=False)
load_dotenv(default_path, override=False)

# Then override with .env.local if present
local_path = find_dotenv(".env.local", raise_error_if_not_found=False)
load_dotenv(local_path, override=True)

# Now environment variables from .env.local are available
print("Debug:", os.getenv("DEBUG"))
```

## requirements.txt

Once you have activated your virtual environment and installed packages via `pip`, you need a way to record exactly which packages (and versions) your project depends on. That’s where `requirements.txt` comes in. It’s simply a plain-text file listing:

```txt
Flask==2.1.2
requests==2.28.1
python-dotenv==1.0.0
```

Using this file, anyone (or any CI/CD pipeline) can install your project’s dependencies with:

```bash
pip install -r requirements.txt
```

This replicates the same package versions you used during development.

How to Generate and Maintain `requirements.txt`

Install everything you need:
```bash
pip install flask requests python-dotenv
```

“Freeze” your environment:

```bash
pip freeze > requirements.txt

```

This writes every installed package in the current venv to `requirements.txt`, pinned to the exact version.

If you omit versions (e.g., Flask instead of Flask==2.1.2), you risk installing a newer or older version that may introduce breaking changes.

```bash
cat requirements.txt 

certifi==2025.4.26
charset-normalizer==2.1.1
click==8.2.1
Flask==2.1.2
idna==3.10
itsdangerous==2.2.0
Jinja2==3.1.6
MarkupSafe==3.0.2
python-dotenv==1.0.0
requests==2.28.1
urllib3==1.26.20
Werkzeug==3.1.3
```

Regularly update as needed:
- If you add a new package (e.g., `pip install sqlalchemy`), run `pip freeze > requirements.txt` again to update.
- If you bump versions, do so deliberately and then re-freeze.

### Best practice in long-term projects:

- **Use two files:**
    - requirements.txt (pins exact versions).
    - requirements.in (lists top-level packages without versions).

Then use a tool like <span class="site-footer-credits"><a href="https://github.com/jazzband/pip-tools"  target="_blank">pip-tools. 

```
# In requirements.in:
flask
requests
python-dotenv

# To compile:
pip-compile requirements.in  # generates a version-pinned requirements.txt
```

This separates “I want Flask” (in .in) from “we tested with Flask 2.1.2” (in .txt), making upgrades more controlled.


## environment.yml

If you need more than pure-Python packages—such as GPU-enabled libraries (`cudatoolkit`), system libraries (`ffmpeg`), or 
non-Python binaries—Conda may be a better choice than `venv`. An `environment.yml` file:

- Specifies both Conda packages and pip packages.
- Can pin versions from specific Conda channels (e.g., conda-forge).
- Exports environment in a single file that includes Python, dependencies, and channels.


Install Miniconda or Anaconda (if not already installed).

```bash
# On Linux/macOS
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
source ~/.bashrc
```

Create and activate a new environment:

```bash
conda create --name myenv python=3.10
conda activate myenv
```

Install packages via Conda or pip:

```bash
conda install numpy pandas python-dotenv
pip install requests flask
```

- Conda can install binary dependencies (e.g., `conda install -c conda-forge ffmpeg`).
- If something is only on PyPI, you can still use `pip install`.

Export to `environment.yml`:

To capture only the packages you explicitly requested (and allow minor version updates), run:

```bash
conda env export --from-history > environment.yml
```

To capture exact versions of every package (including dependencies), omit `--from-history`:

```bash
conda env export > environment.yml

```

Sample `environment.yml` structure:

```yml
name: myenv
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.10
  - numpy
  - pandas
  - python-dotenv
  - pip        # must list pip if you have pip packages
  - pip:
    - requests
    - flask
```

- **name**: (optional) The default environment name when you create it.
- **channels:** The order Conda searches for packages. Placing conda-forge first ensures it picks packages from there in priority.
- **dependencies:**
    - List Conda packages and version constraints (e.g., python=3.10).
    - If any pip packages are required, include pip under dependencies, then indent a pip: section listing those packages.



Anyone with `environment.yml` can run:

```bash
conda env create -f environment.yml
conda activate myenv
```

This installs exactly the same (or, if you used `--from-history`, compatible) versions of Python and all packages listed. 
It’s especially valuable for data science projects where you might need a specific CUDA version or system library.

## Comparing venv + requirements.txt vs. Conda + environment.yml

| Aspect                     | venv + requirements.txt                        | Conda + environment.yml                                    |
|----------------------------|------------------------------------------------|-------------------------------------------------------------|
| Scope                      | Pure Python packages from PyPI                 | Conda (binaries, system libs) + pip                         |
| Non-Python dependencies    | Must install manually (e.g., apt)              | Can include ffmpeg, cudatoolkit directly                     |
| Channels / Repositories    | Only PyPI                                      | Multiple (e.g., conda-forge, defaults)                       |
| Environment isolation      | Python interpreter + site-packages             | Full Conda environment (Python + binaries)                   |
| Reproducibility            | pip freeze pins exact versions                 | Conda pins everything or uses history-based tags             |
| Ease of use                | Built into Python (no extra install)           | Requires installing Conda                                    |
| Package availability       | Only what’s on PyPI                            | Includes packages that don’t exist on PyPI                    |
| File size                  | requirements.txt is small                      | environment.yml can be large if many pins                    |


- If you only need pure-Python libraries (Flask, requests, pandas), venv + requirements.txt is lighter and built-in.
- If you require system-level libraries (OpenCV, TensorFlow with CUDA, ffmpeg), or want GPU-enabled builds, Conda + environment.yml is the way to go.

## Putting It All Together: Sample Project Workflow

Below is an example workflow illustrating how you might set up a new project with both approaches side by side.

Create the project folder and Git repo:

```bash
mkdir my_py_app
cd my_py_app
git init
```

Set up the virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install packages and create requirements.txt:

```bash
pip install flask python-dotenv requests
pip freeze > requirements.txt

```

Create .env for secrets:

```bash
touch .env
nano .env

SECRET_KEY=your-secret-key
DATABASE_URL=postgres://user:pass@localhost:5432/dbname
DEBUG=True

```

Add to .gitignore:


```bash
.venv/
__pycache__/
.env
```

Example main.py:

```python
from dotenv import load_dotenv
import os
from flask import Flask

load_dotenv()

app = Flask(__name__)
app.config["SECRET_KEY"] = os.getenv("SECRET_KEY")

@app.route("/")
def index():
    return f"DB URL: {os.getenv('DATABASE_URL')}<br>Debug: {os.getenv('DEBUG')}"

if __name__ == "__main__":
    app.run(debug=os.getenv("DEBUG", "False").lower() in ("true", "1"))
```


### Conda-Based Project (environment.yml)

Create the project folder and Git repo:

```bash
mkdir my_conda_app
cd my_conda_app
git init
```

Create and activate a Conda environment:

```bash
conda create --name myenv python=3.10
conda activate myenv
```

Install essential packages:

```bash
conda install flask python-dotenv pandas
pip install requests
conda install -c conda-forge ffmpeg  # for example, if you need video processing
```

Create environment.yml:

```bash
conda env export --from-history > environment.yml
```

The file might contain:

```yml
name: myenv
channels:
  - conda-forge
  - defaults
dependencies:
  - python=3.10
  - flask
  - python-dotenv
  - pandas
  - pip
  - pip:
    - requests
  - ffmpeg
```

- Create .env as before and add it to .gitignore.
- Example app.py: (Same as in the venv example.)

**Recreating the environment:** Anyone cloning the repo runs:

```bash
conda env create -f environment.yml
conda activate myenv
```

### Conda Environment Commands

| Command                                                               | Description                                                            |
| --------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `conda create --name myenv`                                           | Create a new environment named `myenv`.                                |
| `conda create --name myenv python=3.10`                               | Create an environment with a specific Python version.                  |
| `conda activate myenv`                                                | Activate an environment.                                               |
| `conda deactivate`                                                    | Deactivate the current environment.                                    |
| `conda env list` or `conda info --envs`                               | List all environments.                                                 |
| `conda remove --name myenv --all`                                     | Delete an environment.                                                 |
| `conda list`                                                          | Show installed packages in the current environment.                    |
| `conda install package_name`                                          | Install a package into the active environment.                         |
| `conda install package_name=1.2.3`                                    | Install a specific package version.                                    |
| `conda update package_name`                                           | Update a specific package.                                             |
| `conda update --all`                                                  | Update all packages in the environment.                                |
| `conda env export > environment.yml`                                  | Export the current environment to a YAML file (includes dependencies). |
| `conda env export --no-builds \| grep -v "prefix:" > environment.yml` | Export without build numbers and prefix (for portability).             |
| `conda env create --file environment.yml`                             | Create an environment from a YAML file.                                |
| `conda env update --file environment.yml`                             | Update the current environment from a YAML file.                       |
| `conda env update --file environment.yml --prune`                     | Update and remove any packages not in the YAML file.                   |
| `conda search package_name`                                           | Search for a package in Conda repositories.                            |
| `conda clean --all`                                                   | Remove unused packages and caches.                                     |


## Conclusion

By following these practices—using `venv` or `Conda`, leveraging `.env` for configuration, and pinning dependencies 
in `requirements.txt` or `environment.yml`—you’ll achieve a reproducible, maintainable, and secure project setup. These conventions scale well whether you’re working solo or collaborating with a team.

---

- [Home](./../../README.md)
- [Python Tutorials](./../tutorials.md)