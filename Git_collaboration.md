# GitHub Collaboration Guide for Lab2

**Repository**: `https://github.com/DanielPhillipsSanchez/Lab2.git`

---

## 1. Git Basics

| Concept | Description |
|---------|-------------|
| **Repository** | Project folder with complete version history |
| **Commit** | Snapshot of changes with a message |
| **Branch** | Isolated line of development |
| **Merge** | Integrating changes between branches |
| **Remote** | Shared repository on GitHub |

---

## 2. Initial Setup

```bash
# Fork the repo on GitHub, then clone your fork
git clone https://github.com/YOUR_USERNAME/Lab2.git
cd Lab2

# Add upstream remote
git remote add upstream https://github.com/DanielPhillipsSanchez/Lab2.git
```

---

## 3. Development Workflow

```bash
# 1. Sync with upstream
git checkout main
git pull upstream main

# 2. Create feature branch
git checkout -b feature/your-feature-name

# 3. Make changes, stage, and commit
git add .
git commit -m "Add your descriptive message"

# 4. Push to your fork
git push origin feature/your-feature-name

# 5. Create Pull Request on GitHub
```

**Branch Prefixes**: `feature/`, `bugfix/`, `hotfix/`, `docs/`

---

## 4. Pull Request Process

1. Push your branch → Click **Compare & pull request** on GitHub
2. Fill in summary of changes
3. Address review feedback
4. After approval → **Squash and merge**
5. Delete branch after merge

---

## 5. Protecting the Main Branch (Step-by-Step)

### Step 5.1: Access Branch Protection Settings

1. Go to your repository: `https://github.com/DanielPhillipsSanchez/Lab2`
2. Click **Settings** (top menu, gear icon)
3. In the left sidebar, click **Branches** (under "Code and automation")
4. Click **Add branch protection rule** (or **Add rule**)

### Step 5.2: Configure the Rule

1. **Branch name pattern**: Type `main`

2. **Protect matching branches** - Enable these options:

   | Setting | Action |
   |---------|--------|
   | **Require a pull request before merging** | Check the box |
   | → Require approvals | Set to `1` (or more) |
   | → Dismiss stale pull request approvals | Check the box |
   | → Require review from Code Owners | Optional |
   | **Require status checks to pass before merging** | Check the box |
   | → Require branches to be up to date | Check the box |
   | **Require conversation resolution before merging** | Check the box |
   | **Require signed commits** | Optional (for verified commits) |
   | **Require linear history** | Optional (forces rebase/squash) |
   | **Do not allow bypassing the above settings** | Check the box |

3. **Rules applied to everyone including administrators** - Enable these:

   | Setting | Action |
   |---------|--------|
   | **Allow force pushes** | Leave UNCHECKED |
   | **Allow deletions** | Leave UNCHECKED |

4. Click **Create** (or **Save changes**)

### Step 5.3: Verify Protection is Active

1. Go back to **Settings → Branches**
2. You should see `main` listed under "Branch protection rules"
3. A lock icon appears next to the branch name in the repository

### Step 5.4: Test the Protection

Try pushing directly to main:
```bash
git checkout main
echo "test" >> test.txt
git add . && git commit -m "Test direct push"
git push origin main
```

Expected result: **Push rejected** with message about protected branch.

---

## 6. CI/CD Integration (Optional)

Create `.github/workflows/ci.yml`:

```yaml
name: CI Pipeline

on:
  pull_request:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: '3.11'
      - run: pip install dbt-snowflake
      - run: cd packages/dbt && dbt compile --profiles-dir .
        env:
          SNOWFLAKE_ACCOUNT: ${{ secrets.SNOWFLAKE_ACCOUNT }}
          SNOWFLAKE_USER: ${{ secrets.SNOWFLAKE_USER }}
          SNOWFLAKE_PASSWORD: ${{ secrets.SNOWFLAKE_PASSWORD }}
```

Add secrets in **Settings → Secrets and variables → Actions**.

---

## 7. Quick Reference

| Task | Command |
|------|---------|
| Create branch | `git checkout -b feature/name` |
| Stage & commit | `git add . && git commit -m "msg"` |
| Push branch | `git push origin feature/name` |
| Sync fork | `git fetch upstream && git merge upstream/main` |
| Delete branch | `git branch -d feature/name` |

---

## 8. Troubleshooting

**Merge conflicts:**
```bash
git fetch upstream
git rebase upstream/main
# Resolve conflicts, then:
git add . && git rebase --continue
git push origin feature/name --force-with-lease
```

**Undo last commit:**
```bash
git reset --soft HEAD~1   # Keep changes
git reset --hard HEAD~1   # Discard changes
```
