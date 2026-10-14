# Python 3.11 Runtime Upgrade Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make Python 3.11 the supported and exercised runtime for local development, Docker, CI, and the backend unit-test command.

**Architecture:** Keep the existing FastAPI/Pydantic v1/SQLAlchemy 1.4 dependency boundaries unchanged. Update only runtime declarations and instructions, create an isolated Python 3.11 environment, install the existing dependency set there, and verify the requested core test without touching frontend or local data directories.

**Tech Stack:** Python 3.11, PowerShell, pytest, Docker, GitHub Actions, `pyproject.toml`.

---

### Task 1: Align repository runtime declarations

**Files:**
- Modify: `Dockerfile`
- Modify: `.github/workflows/ci.yml`
- Modify: `pyproject.toml`
- Modify: `README.md`
- Modify: `README.en.md`
- Modify: `requirements.txt`

- [ ] **Step 1: Update runtime declarations**

Change Docker base images and CI `PYTHON_VERSION` from 3.9 to 3.11. Make `pyproject.toml` require Python 3.11, set Black/Ruff targets to `py311`, and set mypy `python_version` to `3.11`. Update both README installation sections to state Python 3.11 is required. Keep dependency versions unchanged except for Python-version comments.

- [ ] **Step 2: Check for stale Python 3.9/3.10 runtime references**

Run:

```powershell
Select-String -Path 'Dockerfile','.github/workflows/ci.yml','pyproject.toml','README.md','README.en.md','requirements.txt' -Pattern '3\.9|3\.10|Python 3\.10|python_version = "3\.10"|target-version.*py310' -CaseSensitive:$false
```

Expected: no active runtime declaration still selects Python 3.9 or 3.10; historical compatibility wording may be retained only if it explicitly describes non-target compatibility.

### Task 2: Install and isolate Python 3.11 locally

**Files:**
- Create: `.venv311/` (ignored local environment, not committed)

- [ ] **Step 1: Confirm an install route**

Use an available package manager or the official Python 3.11 Windows installer. Do not replace the system Python 3.8 installation and do not modify Anaconda.

- [ ] **Step 2: Create the project environment**

Run from the repository root:

```powershell
py -3.11 -m venv .venv311
```

Expected: `.venv311\Scripts\python.exe --version` reports `Python 3.11.x`.

- [ ] **Step 3: Install project dependencies in the isolated environment**

Run:

```powershell
.venv311\Scripts\python.exe -m pip install --upgrade pip
.venv311\Scripts\python.exe -m pip install -r requirements.txt
.venv311\Scripts\python.exe -m pip install pytest pytest-asyncio
```

Do not use the project database, Chroma persistence, external LLMs, or frontend dependencies for this step.

### Task 3: Verify the requested backend test

**Files:**
- Test: `backend/tests/unit/test_core.py`
- Test target: `backend/core/utils/validators.py` and its imported core utilities

- [ ] **Step 1: Run the exact requested test with Python 3.11**

Run:

```powershell
.venv311\Scripts\python.exe -m pytest backend/tests/unit/test_core.py
```

Expected: collection succeeds under Python 3.11 and the test result is reported with its real exit code. If a dependency or application-import failure remains, document it without bypassing the failure or using real external services.

- [ ] **Step 2: Verify repository scope and generated-file exclusion**

Run:

```powershell
git status --short
```

Expected: only the intended runtime/config/documentation changes are present; `.venv311/`, databases, Chroma data, frontend files, Alembic files, and Docker Compose are not modified or tracked.

### Task 4: Final verification

- [ ] **Step 1: Check syntax/configuration files**

Run:

```powershell
.venv311\Scripts\python.exe -m py_compile backend/core/utils/validators.py backend/tests/unit/test_core.py
```

Expected: exit code 0.

- [ ] **Step 2: Report runtime and test evidence**

Report the Python executable path, Python 3.11 version, exact test command, real test exit code, changed files, and any unrun Docker/CI checks.
