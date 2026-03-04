# GitHub Collaboration Guide for Lab2

This guide provides a structured approach to integrating GitHub collaboration practices into the **Lab2** project (Performance Intelligence).

**Repository**: `https://github.com/DanielPhillipsSanchez/Lab2.git`

---

## Phase 1: Understanding Git Fundamentals

### What is Git?

Git is a distributed version control system that tracks changes in source code during software development. It enables multiple developers to work on the same codebase simultaneously without overwriting each other's work.

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Repository** | A directory containing all project files and the complete revision history |
| **Commit** | A snapshot of changes at a specific point in time with a descriptive message |
| **Branch** | An independent line of development that isolates work from the main codebase |
| **Merge** | The process of integrating changes from one branch into another |
| **Remote** | A shared repository hosted on a server (GitHub) that team members push to and pull from |

---

## Phase 2: Setting Up Your Environment

### Step 2.1: Fork the Repository

Forking creates your personal copy of the Lab2 repository under your GitHub account.

1. Navigate to `https://github.com/DanielPhillipsSanchez/Lab2`
2. Click the **Fork** button (top-right corner)
3. Select your account as the destination
4. Wait for the fork to complete

### Step 2.2: Clone Your Fork Locally

```bash
# Clone your forked repository
git clone https://github.com/YOUR_USERNAME/Lab2.git

# Navigate into the project directory
cd Lab2

# Add the original repository as upstream remote
git remote add upstream https://github.com/DanielPhillipsSanchez/Lab2.git

# Verify remotes are configured correctly
git remote -v
```

Expected output:
```
origin    https://github.com/YOUR_USERNAME/Lab2.git (fetch)
origin    https://github.com/YOUR_USERNAME/Lab2.git (push)
upstream  https://github.com/DanielPhillipsSanchez/Lab2.git (fetch)
upstream  https://github.com/DanielPhillipsSanchez/Lab2.git (push)
```

### Step 2.3: Keep Your Fork Synchronized

```bash
# Fetch latest changes from upstream
git fetch upstream

# Switch to your main branch
git checkout main

# Merge upstream changes into your local main
git merge upstream/main

# Push updates to your fork
git push origin main
```

---

## Phase 3: Feature Development Workflow

### Step 3.1: Create a Feature Branch

Always create a new branch for each feature or bug fix. Never work directly on `main`.

```bash
# Ensure you're on main and it's up to date
git checkout main
git pull upstream main

# Create and switch to a new feature branch
git checkout -b feature/your-feature-name
```

**Branch Naming Conventions:**

| Prefix | Purpose | Example |
|--------|---------|---------|
| `feature/` | New functionality | `feature/add-chargeback-metrics` |
| `bugfix/` | Bug repairs | `bugfix/fix-approval-rate-calc` |
| `hotfix/` | Urgent production fixes | `hotfix/settlement-null-check` |
| `docs/` | Documentation updates | `docs/update-api-reference` |
| `refactor/` | Code restructuring | `refactor/optimize-deposits-query` |

### Step 3.2: Make Your Changes

1. Implement your changes following project conventions
2. Stage modified files:
   ```bash
   # Stage specific files
   git add packages/dbt/models/marts/your_model.sql

   # Or stage all changes
   git add .
   ```
3. Commit with a clear message:
   ```bash
   git commit -m "Add chargeback win rate metric to MARTS layer"
   ```

**Commit Message Guidelines:**

- Use imperative mood: "Add feature" not "Added feature"
- Keep the first line under 72 characters
- Reference issue numbers when applicable: `Fix #42: Resolve settlement amount mismatch`
- Be specific about what changed and why

### Step 3.3: Push Your Branch

```bash
git push origin feature/your-feature-name
```

---

## Phase 4: Pull Request Process

### Step 4.1: Create a Pull Request

1. Navigate to your fork on GitHub
2. Click **Compare & pull request** (appears after pushing)
3. Ensure the base repository is `DanielPhillipsSanchez/Lab2` and base branch is `main`
4. Fill in the PR template:

```markdown
## Summary
Brief description of what this PR accomplishes.

## Changes Made
- Added X to handle Y
- Modified Z to improve performance
- Updated documentation for A

## Testing
- [ ] dbt models compile successfully
- [ ] All tests pass
- [ ] Manual verification completed

## Related Issues
Closes #123
```

### Step 4.2: Respond to Review Feedback

1. Address reviewer comments by making additional commits
2. Request re-review after changes are complete
3. Resolve conversations as feedback is addressed
4. Keep the PR focused on a single objective

### Step 4.3: Merge and Cleanup

After approval:

1. Maintainer merges the PR using **Squash and merge** (preferred)
2. Delete your feature branch:
   ```bash
   # Delete local branch
   git branch -d feature/your-feature-name

   # Delete remote branch
   git push origin --delete feature/your-feature-name
   ```

---

## Phase 5: Branch Protection Configuration

### Step 5.1: Enable Branch Protection Rules

Navigate to **Settings → Branches → Add branch protection rule** for the `main` branch.

### Step 5.2: Required Protection Settings

| Setting | Value | Purpose |
|---------|-------|---------|
| Branch name pattern | `main` | Protects the main branch |
| Require pull request before merging | Enabled | Forces code review |
| Required approvals | 1-2 | Ensures peer review |
| Dismiss stale approvals | Enabled | Re-review after changes |
| Require status checks | Enabled | CI must pass |
| Require up-to-date branches | Enabled | Prevents merge conflicts |
| Require conversation resolution | Enabled | All comments addressed |
| Include administrators | Enabled | No bypass for anyone |
| Allow force pushes | Disabled | Protects commit history |
| Allow deletions | Disabled | Prevents accidental deletion |

### Step 5.3: Configure Required Status Checks

Add these status checks (if CI/CD is configured):

- `dbt build`
- `dbt test`
- `lint`

---

## Phase 6: CI/CD Integration

### Step 6.1: Create GitHub Actions Workflow

Create `.github/workflows/ci.yml`:

```yaml
name: CI Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dbt
        run: pip install dbt-snowflake

      - name: Compile dbt models
        run: |
          cd packages/dbt
          dbt compile --profiles-dir .
        env:
          SNOWFLAKE_ACCOUNT: ${{ secrets.SNOWFLAKE_ACCOUNT }}
          SNOWFLAKE_USER: ${{ secrets.SNOWFLAKE_USER }}
          SNOWFLAKE_PASSWORD: ${{ secrets.SNOWFLAKE_PASSWORD }}

      - name: Run dbt tests
        run: |
          cd packages/dbt
          dbt test --profiles-dir .
        env:
          SNOWFLAKE_ACCOUNT: ${{ secrets.SNOWFLAKE_ACCOUNT }}
          SNOWFLAKE_USER: ${{ secrets.SNOWFLAKE_USER }}
          SNOWFLAKE_PASSWORD: ${{ secrets.SNOWFLAKE_PASSWORD }}
```

### Step 6.2: Configure Repository Secrets

Navigate to **Settings → Secrets and variables → Actions** and add:

| Secret Name | Description |
|-------------|-------------|
| `SNOWFLAKE_ACCOUNT` | Snowflake account identifier |
| `SNOWFLAKE_USER` | Service account username |
| `SNOWFLAKE_PASSWORD` | Service account password |
| `SNOWFLAKE_ROLE` | Role for CI operations |
| `SNOWFLAKE_WAREHOUSE` | Warehouse for CI jobs |

---

## Phase 7: Code Review Best Practices

### For Authors

1. **Keep PRs small** - Aim for under 400 lines of changes
2. **Self-review first** - Check your own code before requesting review
3. **Provide context** - Explain the "why" in your PR description
4. **Test thoroughly** - Verify all changes work as expected
5. **Be responsive** - Address feedback promptly

### For Reviewers

1. **Be constructive** - Focus on improvement, not criticism
2. **Ask questions** - Clarify intent before suggesting changes
3. **Prioritize feedback** - Distinguish blocking issues from suggestions
4. **Acknowledge good work** - Recognize well-written code
5. **Review promptly** - Respond within 24-48 hours

### Review Checklist

- [ ] Code follows project conventions and style
- [ ] SQL queries are optimized and follow Snowflake best practices
- [ ] dbt models include appropriate tests and documentation
- [ ] No sensitive data (credentials, PII) is exposed
- [ ] Changes are backward compatible
- [ ] Error handling is appropriate

---

## Phase 8: Troubleshooting Common Issues

### Merge Conflicts

```bash
# Fetch latest changes
git fetch upstream

# Rebase your branch on main
git rebase upstream/main

# Resolve conflicts in your editor, then continue
git add .
git rebase --continue

# Force push to update your PR
git push origin feature/your-feature-name --force-with-lease
```

### Undoing Changes

```bash
# Discard uncommitted changes to a file
git checkout -- path/to/file

# Undo the last commit (keep changes staged)
git reset --soft HEAD~1

# Undo the last commit (discard changes)
git reset --hard HEAD~1
```

### Syncing a Stale Fork

```bash
git fetch upstream
git checkout main
git reset --hard upstream/main
git push origin main --force
```

---

## Quick Reference Card

| Task | Command |
|------|---------|
| Create branch | `git checkout -b feature/name` |
| Stage changes | `git add .` |
| Commit | `git commit -m "message"` |
| Push branch | `git push origin feature/name` |
| Sync with upstream | `git fetch upstream && git merge upstream/main` |
| Delete local branch | `git branch -d feature/name` |
| Delete remote branch | `git push origin --delete feature/name` |
| View branch status | `git status` |
| View commit history | `git log --oneline -10` |

---

## Project-Specific Notes

### Lab2 Directory Structure

```
Lab2/
├── packages/
│   ├── database/         # SQL deployment scripts
│   │   └── utilities/    # Agent and semantic view definitions
│   └── dbt/              # dbt transformation project
│       ├── models/
│       │   ├── staging/      # Views over RAW data
│       │   ├── intermediate/ # Enriched dynamic tables
│       │   └── marts/        # Business-ready tables
│       └── analyses/         # Semantic view DDL
├── apps/                 # Application code
├── AGENTS.md             # Project context for AI agents
└── setup.sql             # Initial database setup
```

### Key Files to Review Before Contributing

1. `AGENTS.md` - Project architecture and business rules
2. `packages/dbt/dbt_project.yml` - dbt configuration
3. `packages/dbt/models/` - Existing model patterns

---

## Contact and Support

For questions or issues with the collaboration process, create an issue in the repository or contact the project maintainer.
