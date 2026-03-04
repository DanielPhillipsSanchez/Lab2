# GitHub Repository Management Guide (Code Owners)

This guide is for **repository administrators and code owners** responsible for setting up and managing the Lab2 project.

**Repository**: `https://github.com/DanielPhillipsSanchez/Lab2.git`

---

## 1. Creating a New Repository

### Step 1.1: Create Repository on GitHub

1. Go to [github.com](https://github.com) and sign in
2. Click the **+** icon (top-right) → **New repository**
3. Configure:
   - **Repository name**: `Lab2`
   - **Description**: "Performance Intelligence - Payment Analytics Platform"
   - **Visibility**: Private (recommended) or Public
   - **Initialize**: Check "Add a README file"
   - **Add .gitignore**: Select `Node` or `Python` template
4. Click **Create repository**

### Step 1.2: Clone and Initialize Locally

```bash
git clone https://github.com/YOUR_USERNAME/Lab2.git
cd Lab2
```

---

## 2. Granting Access to Collaborators

### Step 2.1: Add Individual Collaborators

1. Go to **Settings → Collaborators and teams** (left sidebar under "Access")
2. Click **Add people**
3. Search by GitHub username or email
4. Select the appropriate role:

| Role | Permissions |
|------|-------------|
| **Read** | View and clone repository |
| **Triage** | Read + manage issues and PRs |
| **Write** | Triage + push to non-protected branches |
| **Maintain** | Write + manage settings (except destructive) |
| **Admin** | Full access including dangerous operations |

5. Click **Add [username] to this repository**

### Step 2.2: Create and Manage Teams (Organizations)

For organization repositories:

1. Go to **Organization Settings → Teams**
2. Click **New team**
3. Name the team (e.g., `lab2-developers`, `lab2-reviewers`)
4. Add members to the team
5. Go to **Repository Settings → Collaborators and teams**
6. Click **Add teams** and assign repository permissions

### Step 2.3: Recommended Team Structure

| Team | Role | Purpose |
|------|------|---------|
| `lab2-admins` | Admin | Repository owners and maintainers |
| `lab2-reviewers` | Maintain | Code review and PR approval |
| `lab2-developers` | Write | Active contributors |
| `lab2-readonly` | Read | Observers and stakeholders |

---

## 3. Protecting the Main Branch

### Step 3.1: Access Branch Protection

1. Go to **Settings → Branches** (under "Code and automation")
2. Click **Add branch protection rule**

### Step 3.2: Configure Protection Rules

1. **Branch name pattern**: `main`

2. Enable these settings:

| Setting | Configuration |
|---------|---------------|
| **Require a pull request before merging** | ✓ Enable |
| → Require approvals | `1` or `2` |
| → Dismiss stale pull request approvals when new commits are pushed | ✓ Enable |
| → Require review from Code Owners | ✓ Enable (if CODEOWNERS file exists) |
| **Require status checks to pass before merging** | ✓ Enable |
| → Require branches to be up to date before merging | ✓ Enable |
| **Require conversation resolution before merging** | ✓ Enable |
| **Do not allow bypassing the above settings** | ✓ Enable |
| **Restrict who can push to matching branches** | Optional - limit to specific teams |
| **Allow force pushes** | ✗ Disable |
| **Allow deletions** | ✗ Disable |

3. Click **Create**

### Step 3.3: Create CODEOWNERS File

Create `.github/CODEOWNERS` to automatically request reviews:

```
# Default owners for everything
* @DanielPhillipsSanchez

# dbt models require data team review
/packages/dbt/ @DanielPhillipsSanchez

# Database scripts require admin review
/packages/database/ @DanielPhillipsSanchez

# Apps require frontend team review
/apps/ @DanielPhillipsSanchez
```

---

## 4. Repository Settings Configuration

### Step 4.1: General Settings

Go to **Settings → General**:

| Setting | Recommended Value |
|---------|-------------------|
| **Default branch** | `main` |
| **Features** | Enable Issues, Projects as needed |
| **Pull Requests** | ✓ Allow squash merging (preferred) |
| | ✓ Allow merge commits |
| | ✗ Allow rebase merging (optional) |
| | ✓ Automatically delete head branches |

### Step 4.2: Configure Merge Button

Under **Pull Requests**:
- **Default to PR title for squash commits**: Enabled
- **Suggest updating PR branches**: Enabled

---

## 5. Setting Up CI/CD

### Step 5.1: Add Repository Secrets

1. Go to **Settings → Secrets and variables → Actions**
2. Click **New repository secret**
3. Add required secrets:

| Secret | Description |
|--------|-------------|
| `SNOWFLAKE_ACCOUNT` | Snowflake account identifier |
| `SNOWFLAKE_USER` | Service account username |
| `SNOWFLAKE_PASSWORD` | Service account password |
| `SNOWFLAKE_ROLE` | CI role (e.g., `TRANSFORM_ROLE`) |
| `SNOWFLAKE_WAREHOUSE` | CI warehouse |

### Step 5.2: Create CI Workflow

Create `.github/workflows/ci.yml`:

```yaml
name: CI

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
      - run: cd packages/dbt && dbt compile
        env:
          SNOWFLAKE_ACCOUNT: ${{ secrets.SNOWFLAKE_ACCOUNT }}
          SNOWFLAKE_USER: ${{ secrets.SNOWFLAKE_USER }}
          SNOWFLAKE_PASSWORD: ${{ secrets.SNOWFLAKE_PASSWORD }}
```

---

## 6. Monitoring and Maintenance

### Step 6.1: Review Access Regularly

1. Go to **Settings → Collaborators and teams**
2. Remove users who no longer need access
3. Audit admin permissions quarterly

### Step 6.2: Monitor Branch Protection

1. Go to **Settings → Branches**
2. Verify rules are active (lock icon visible)
3. Review bypass attempts in **Security → Audit log** (organizations)

### Step 6.3: Manage Pending Invitations

1. Go to **Settings → Collaborators and teams**
2. Check **Pending invitations** section
3. Resend or cancel stale invitations

---

## 7. Quick Reference

| Task | Location |
|------|----------|
| Add collaborators | Settings → Collaborators and teams |
| Protect branches | Settings → Branches |
| Add secrets | Settings → Secrets and variables → Actions |
| View audit log | Settings → Security → Audit log |
| Manage webhooks | Settings → Webhooks |
| Configure Pages | Settings → Pages |

---

## 8. Troubleshooting

**Collaborator can't push:**
- Verify they have Write access or higher
- Check branch protection isn't blocking their push
- Ensure they're pushing to a feature branch, not `main`

**PR can't be merged:**
- Check required status checks are passing
- Verify required approvals are met
- Ensure conversations are resolved

**CODEOWNERS not working:**
- File must be in `.github/CODEOWNERS`, `CODEOWNERS`, or `docs/CODEOWNERS`
- Users must have Write access to be valid code owners
- Check file syntax (no trailing spaces)
