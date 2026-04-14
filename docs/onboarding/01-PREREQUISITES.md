# 01 — Prerequisites and Environment Setup

> **Audience**: Principal Investigators new to OMAIB
> **Time to complete**: 15–20 minutes
> **Before you start**: Read [00 — Overview of OMAIB](00-OVERVIEW.md) to understand the OMAIB model
> **Next step**: [02 — Creating Your Domain Adapter Pack](02-CREATE-ADAPTER.md)

---

## What This Document Covers

Before you can create a Domain Adapter Pack (DAP), you need four things in place:

1. A GitHub repository to host your adapter
2. Python 3.11 or later installed
3. The `omaib-contracts` package installed
4. Git configured on your machine

This document walks through each requirement with verification commands so you know exactly when you're ready to proceed.

---

## 1. GitHub Repository

Your adapter files live in a folder called `omaib-adapter/` inside **your own GitHub repository**. OMAIB does not host your data or your code — it registers a pointer to your repo and pulls metadata from it.

### Requirements

| Requirement | Notes |
|---|---|
| A GitHub account | Free tier is fine |
| A repository (new or existing) | Public or private both work |
| Push access to the `main` or `master` branch | You need to be able to push commits |

### Creating a repository (if you don't have one yet)

1. Go to [github.com/new](https://github.com/new)
2. Choose a name that reflects your research project — e.g., `omaib-benchmark`, `litbench`
3. Set visibility to **Public** (highly recommended) or **Private** (if necessary)
4. Initialise with a README — this avoids an empty repo error later
5. Click **Create repository**

> **Naming tip**: Your repository name will eventually appear in the OMAIB registry. Keep it lowercase with hyphens, no spaces, no underscores.

### Clone the repository locally

```bash
git clone https://github.com/<your-org>/<your-repo>.git
cd <your-repo>
```

---

## 2. Python 3.11 or Later

The `omaib-contracts` package requires Python 3.11+. Python 3.12 is recommended.

### Check your Python version

```bash
python --version
# or, on systems where python3 is separate:
python3 --version
```

You should see something like:

```text
Python 3.12.3
```

If you see Python 3.10 or earlier, you need to upgrade before continuing.

### Installing Python

| Platform | Recommended method |
|---|---|
| **macOS** | `brew install <python@3.1>2` or download from [python.org](https://www.python.org/downloads/) |
| **Ubuntu / Debian** | `sudo apt install python3.12 python3.12-venv python3.12-pip` |
| **Windows** | Download from [python.org](https://www.python.org/downloads/) — tick "Add to PATH" during install |

### Using a Virtual Environment (Recommended)

Working inside a virtual environment keeps your project dependencies isolated:

```bash
# Create the virtual environment inside your adapter repo
python3 -m venv .venv

# Activate it
# macOS / Linux:
source .venv/bin/activate
# Windows PowerShell:
.\.venv\Scripts\Activate.ps1
# Windows CMD:
.venv\Scripts\activate.bat
```

Add `.venv/` to your `.gitignore`:

```bash
echo ".venv/" >> .gitignore
```

---

## 3. Installing omaib-contracts

The `omaib-contracts` package provides:

- **`omaib-init-adapter`** — scaffolds the four required files from templates
- **`omaib-validate-adapter`** — validates your files against OMAIB schemas
- **`omaib-gate-status`** — shows your current gate score and what's blocking progress

### Install from the GitHub repository

```bash
pip install git+https://github.com/omaib/omaib-contracts.git
# OR,
pip install omaib-contracts
```

> **Note**: Until `omaib-contracts` is published on PyPI, installation is via the git URL. The command above always installs the latest stable version from `main`.

### Verify the install

After installation, all three CLI tools should be available:

```bash
omaib-init-adapter --help
omaib-validate-adapter --help
omaib-gate-status --help
```

Each command should print a short help message. If you see `command not found`, check that your virtual environment is activated and that the install completed without errors.

### Upgrading

When OMAIB publishes a schema or template update, reinstall with:

```bash
pip install --upgrade git+https://github.com/omaib/omaib-contracts.git
```

You can check the currently installed version with:

```bash
pip show omaib-contracts
```

---

## 4. Git Configuration

Your commits to the adapter repo are how OMAIB tracks version history. Make sure Git knows who you are:

```bash
git config --global user.name "Your Name"
git config --global user.email "<you@example.com>"
```

Verify:

```bash
git config --list | grep user
```

---

## 5. Optional But Recommended

These are not required for Gate 1, but they make the workflow smoother:

| Tool | Why it helps |
|---|---|
| **VS Code** with the YAML extension | Schema hints and inline validation while editing `.yaml` files |
| **GitHub CLI (`gh`)** | Create issues and releases from the terminal — useful for the registration step |
| **`jq`** | Parses the JSON validation report output from `omaib-validate-adapter --json` |

Install GitHub CLI:

```bash
# macOS
brew install gh
# Ubuntu
sudo apt install gh
# Windows
winget install GitHub.cli
```

---

## Pre-flight Checklist

Run through this before moving to the next step:

- [ ] GitHub repository created and cloned locally
- [ ] `python --version` returns 3.11 or higher
- [ ] Virtual environment created and activated
- [ ] `pip install git+https://github.com/omaib/omaib-contracts.git` completed without errors
- [ ] `omaib-init-adapter --help` prints a help message
- [ ] `git config user.name` returns your name

If all six boxes are checked, you're ready to scaffold your adapter.

---

## Next

→ [02 — Creating Your Domain Adapter Pack](02-CREATE-ADAPTER.md)
