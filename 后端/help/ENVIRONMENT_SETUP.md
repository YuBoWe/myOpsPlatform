# Python Virtual Environment Setup & Troubleshooting Guide

This document explains how to install Python on macOS, manage multiple versions, and handle virtual environments for `proj2`.

## 1. Installing Python on macOS
The best way to install and manage Python on macOS is using **Homebrew**.

### Step 1: Install Homebrew (if not installed)
Check if you have Homebrew installed:
```bash
brew --version
```
If not, visit [brew.sh](https://brew.sh) for the installation command.

### Step 2: Install Python
You can install the latest Python or a specific version.

**Install latest version:**
```bash
brew install python
```

**Install a specific version (e.g., Python 3.10):**
```bash
brew install python@3.10
```

## 2. Managing Multiple Versions
It's common to have multiple Python versions (e.g., system python, python3.11, python3.10).

### Checking Installed Versions
See what versions Homebrew has installed:
```bash
ls /opt/homebrew/opt/python*
```
Or check where your `python3` command points:
```bash
which python3
ls -l $(which python3)
```

### Switching Versions
You generally don't "switch" the global python version. Instead, you **invoke the specific version** you need.

*   To run Python 3.11: `python3.11`
*   To run Python 3.10: `python3.10`

**Example:**
Creating a virtual environment with a specific version:
```bash
# Create a venv using Python 3.10 explicitly
python3.10 -m venv .venv
```

## 3. Creating a Virtual Environment
Once you have your desired Python version, create the environment for the project.

### Creation Command
Run this in the root of your project (`proj2`):
```bash
# Uses your default python3
python3 -m venv .venv

# OR use a specific version
python3.10 -m venv .venv
```
This creates a hidden folder named `.venv` containing an isolated Python runtime.

## 4. Why use a Virtual Environment?
A virtual environment (VSenv) creates an isolated space for your project. This ensures that:
- You can install specific package versions for `proj2` without affecting other projects.
- You avoid permission errors (like `EACCES`) often seen when installing to the global system.

## 5. The "Missing Module" Problem
**Symptom:**
You run a command like `python manage.py startapp ...` and get:
```
ModuleNotFoundError: No module named 'django'
```
Or you check `pip list` and see different packages than you expect.

**Cause:**
This happens because the terminal is using the **Global** Python interpreter (e.g., `/opt/homebrew/bin/python3`) instead of the **Project** interpreter inside `.venv`. The global environment doesn't have your project-specific packages installed.

**Evidence:**
- Global pip: `/opt/homebrew/bin/pip`
- Project pip: `/Volumes/T7/work/proj2/.venv/lib/python3.10/site-packages/pip`

## 6. How to Fix (Activate the Environment)
Before working on the project, you must "activate" the virtual environment in your terminal.

### Activation Command (macOS/Linux)
Run this inside the `proj2` directory:
```bash
source .venv/bin/activate
```
*Note: You should see `(.venv)` appear in your terminal prompt.*

### Verification
After activation, check that you are using the correct tools:
```bash
which python
# Should output: .../proj2/.venv/bin/python

which pip
# Should output: .../proj2/.venv/bin/pip
```

## 7. Managing Packages (pip)
Once activated, all `pip install` commands will affect only this project.

### Check Installed Packages
```bash
pip list
```

### Install New Packages
```bash
pip install package_name
# Example: pip install mysqlclient
```

### Upgrading pip
If you see a notice that a new release is available:
```bash
pip install --upgrade pip
```

## Summary Checklist
1. [ ] Install Python via Homebrew: `brew install python`
2. [ ] Open Terminal in project folder.
3. [ ] (First time only) Run `python3 -m venv .venv` (or `python3.10 -m ...`) to create environment.
4. [ ] Run `source .venv/bin/activate`.
5. [ ] Verify with `which python`.
6. [ ] Run your commands (e.g., `python manage.py runserver`).
