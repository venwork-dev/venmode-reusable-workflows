# Gitleaks Reusable Workflow Setup Guide

## Overview

This setup allows you to maintain a single Gitleaks secret scanning workflow that can be used across all your repositories. Changes to the workflow only need to be made in one place.

## Why Gitleaks?

- ✅ **Completely Free & Open Source** - No cost for personal or organization accounts
- ✅ **Detects 160+ secret types** - API keys, tokens, passwords, certificates, etc.
- ✅ **Active Development** - Regular updates and new secret patterns
- ✅ **GitHub Actions Integration** - Official action available
- ✅ **Free Organization License** - Just need to obtain it for GitHub organizations

## Setup Steps

### Step 1: Create Central Workflow Repository

You have two options:

**Option A: Use `.github` repository (Recommended for Organizations)**
- Create a repository named `.github` in your organization
- This is a special repository that GitHub recognizes for shared workflows

**Option B: Use a dedicated repository**
- Create a repository like `github-workflows` or `shared-workflows`

### Step 2: Add the Reusable Workflow

1. In your central repository, create the directory structure:
   ```
   .github/workflows/
   ```

2. Add the file `reusable-gitleaks-workflow.yml` (provided) to `.github/workflows/`

3. Commit and push to the `main` branch

### Step 3: Configure Repository Access (For Private Repos)

If your central workflow repository is **private**:

1. Go to the central repository → Settings → Actions → General
2. Under "Access", select one of:
   - **"Accessible from repositories in the [YOUR-ORG] organization"** (recommended)
   - Or specify individual repositories

### Step 4: Add Workflow to Each Project Repository

1. In each project repository, create `.github/workflows/secret-scanning.yml`

2. Add the caller workflow content (provided in `caller-workflow-template.yml`)

3. **Important**: Replace the placeholders:
   ```yaml
   uses: YOUR-ORG/YOUR-WORKFLOWS-REPO/.github/workflows/reusable-gitleaks-workflow.yml@main
   ```

   Examples:
   - For `.github` repo: `my-org/.github/.github/workflows/reusable-gitleaks-workflow.yml@main`
   - For custom repo: `my-org/github-workflows/.github/workflows/reusable-gitleaks-workflow.yml@main`

4. Commit and push

### Step 5: Obtain Free Gitleaks License (For Organizations)

**Only required if scanning GitHub Organization repositories:**

1. Visit: https://gitleaks.io/
2. Request a free organization license
3. Once received, add it to your repository secrets:
   - Go to repository Settings → Secrets and variables → Actions
   - Add secret named `GITLEAKS_LICENSE` with your license key
   - Or add it as an organization secret to share across all repos

**Note**: Personal GitHub accounts don't need a license.

## How It Works

### On Every Pull Request:
- Scans only the **changes in the PR** (fast)
- Comments on the PR if secrets are detected
- Blocks merging if secrets are found (you can configure this)

### Daily at Midnight (00:00 UTC):
- Scans the **entire repository** (comprehensive)
- Catches any secrets that may have been in the repo before the workflow was added
- Uploads a report if issues are found

### Manual Trigger:
- You can run a full scan anytime from the Actions tab

## Customization Options

### 1. Custom Gitleaks Configuration

Create a `.gitleaks.toml` file in your repository to:
- Add custom secret patterns
- Exclude specific files or paths
- Configure allowlists

Then update the caller workflow:
```yaml
uses: YOUR-ORG/YOUR-WORKFLOWS-REPO/.github/workflows/reusable-gitleaks-workflow.yml@main
with:
  scan_type: 'pr'
  config_path: '.gitleaks.toml'  # Add this line
```

### 2. Change Schedule

Modify the cron expression in the caller workflow:
```yaml
schedule:
  - cron: '0 0 * * *'  # Daily at midnight
  # Examples:
  # - cron: '0 0 * * 0'  # Weekly on Sunday
  # - cron: '0 0 1 * *'  # Monthly on the 1st
```

### 3. Protect Branches

In your repository settings:
1. Go to Settings → Branches → Add rule
2. Enable "Require status checks to pass before merging"
3. Select "Gitleaks Secret Scanner"

### 4. Version Pinning (Recommended for Production)

Instead of `@main`, pin to a specific tag or commit:
```yaml
uses: YOUR-ORG/YOUR-WORKFLOWS-REPO/.github/workflows/reusable-gitleaks-workflow.yml@v1.0.0
```

## Advantages of This Approach

✅ **Single Source of Truth** - Update once, applies everywhere
✅ **Consistent Security** - All repos use the same scanning logic
✅ **Easy Maintenance** - Fix bugs or update versions in one place
✅ **Reduced Duplication** - No copy-pasting workflows across repos
✅ **Versioning** - Tag releases of your workflow for stability

## What Gitleaks Detects

- API Keys (AWS, Google Cloud, Azure, etc.)
- OAuth Tokens
- Database Credentials
- Private Keys (SSH, RSA, etc.)
- JWT Tokens
- Webhook URLs with secrets
- Password patterns
- Generic secret patterns
- And 160+ more types

## Troubleshooting

### "Workflow not found" error
- Check that the path to the reusable workflow is correct
- Verify the workflow file exists on the `main` branch
- For private repos, check access settings

### License errors (Organizations only)
- Obtain free license from https://gitleaks.io/
- Add as `GITLEAKS_LICENSE` secret in repository or organization settings

### False positives
- Create a `.gitleaks.toml` config file
- Add patterns to the allowlist
- See: https://github.com/gitleaks/gitleaks#configuration

### Workflow doesn't run on PRs
- Check that the branch names match your repository's default branch
- Verify the workflow file is in `.github/workflows/` directory

## Resources

- Gitleaks Documentation: https://github.com/gitleaks/gitleaks
- GitHub Reusable Workflows: https://docs.github.com/en/actions/using-workflows/reusing-workflows
- Gitleaks Action: https://github.com/gitleaks/gitleaks-action

## Example Repository Structure

```
your-org/
├── .github/                    (central workflow repo)
│   └── .github/
│       └── workflows/
│           └── reusable-gitleaks-workflow.yml
│
├── project-1/                  (your project repo)
│   └── .github/
│       └── workflows/
│           └── secret-scanning.yml  (calls central workflow)
│
└── project-2/                  (your project repo)
    └── .github/
        └── workflows/
            └── secret-scanning.yml  (calls central workflow)
```

## Next Steps

1. Create your central workflow repository
2. Add the reusable workflow
3. Add the caller workflow to one test repository
4. Create a test PR with a fake API key to verify it works
5. Roll out to all repositories
6. Sleep better knowing your secrets are protected! 🔒
