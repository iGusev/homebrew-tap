# Creating the GitHub Repository for Homebrew Tap

This guide walks you through creating the GitHub repository for the GLF Homebrew tap.

## Step 1: Create GitHub Repository

### Option A: Using GitHub CLI (Recommended)

```bash
# Navigate to the homebrew-tap directory
cd /Users/igusev/projects/glf-project/glf/homebrew-tap

# Create the repository on GitHub
gh repo create igusev/homebrew-tap \
  --public \
  --description "Homebrew tap for GLF - GitLab Fuzzy Finder" \
  --homepage "https://github.com/igusev/glf" \
  --source=. \
  --remote=origin \
  --push
```

### Option B: Using GitHub Web Interface

1. Go to https://github.com/new
2. Fill in the repository details:
   - **Owner:** igusev
   - **Repository name:** homebrew-tap
   - **Description:** Homebrew tap for GLF - GitLab Fuzzy Finder
   - **Visibility:** Public
   - **Initialize:** Do NOT initialize with README, .gitignore, or license (we already have them)
3. Click "Create repository"

4. Then push your local repository:
   ```bash
   cd /Users/igusev/projects/glf-project/glf/homebrew-tap
   git remote add origin git@github.com:igusev/homebrew-tap.git
   git branch -M main
   git push -u origin main
   ```

## Step 2: Configure GitHub Token for GoReleaser

### Create Personal Access Token

1. Go to https://github.com/settings/tokens/new
2. Configure the token:
   - **Note:** HOMEBREW_TAP_GITHUB_TOKEN
   - **Expiration:** No expiration (or set a long expiration)
   - **Select scopes:**
     - ✅ `repo` (Full control of private repositories)
       - This includes: repo:status, repo_deployment, public_repo, repo:invite, security_events
3. Click "Generate token"
4. **IMPORTANT:** Copy the token immediately - you won't be able to see it again

### Add Token to GLF Repository Secrets

1. Go to https://github.com/igusev/glf/settings/secrets/actions
2. Click "New repository secret"
3. Fill in:
   - **Name:** `HOMEBREW_TAP_GITHUB_TOKEN`
   - **Secret:** Paste the token you copied
4. Click "Add secret"

## Step 3: Update GLF Release Workflow

Update the `.github/workflows/release.yml` file in the glf repository to use the new token:

```yaml
- name: Run GoReleaser
  uses: goreleaser/goreleaser-action@v6
  with:
    distribution: goreleaser
    version: latest
    args: release --clean
  env:
    GITHUB_TOKEN: ${{ secrets.HOMEBREW_TAP_GITHUB_TOKEN }}
```

**Before:**
```yaml
env:
  GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**After:**
```yaml
env:
  GITHUB_TOKEN: ${{ secrets.HOMEBREW_TAP_GITHUB_TOKEN }}
```

## Step 4: Verify Configuration

### Check Repository Settings

1. Go to https://github.com/igusev/homebrew-tap
2. Verify:
   - ✅ Repository is public
   - ✅ Default branch is `main`
   - ✅ Repository has proper description
   - ✅ Files are visible: README.md, Formula/glf.rb, SETUP.md, LICENSE

### Test the Tap Locally

```bash
# Add the tap
brew tap igusev/tap

# Verify it was added
brew tap

# Try to install (will use the formula from your tap)
brew install glf

# Check the version
glf --help
```

### Verify Formula

```bash
# Audit the formula for issues
brew audit --strict igusev/tap/glf

# Test installation from source
brew install --build-from-source igusev/tap/glf
```

## Step 5: Test Release Process

### Create a Test Release

```bash
# Navigate to glf repository
cd /Users/igusev/projects/glf-project/glf

# Create a new tag (use a test version)
git tag -a v0.2.1 -m "Test release for Homebrew tap"
git push origin v0.2.1
```

### Monitor the Release

1. Go to https://github.com/igusev/glf/actions
2. Watch the "Release" workflow execution
3. Check the logs to ensure:
   - ✅ Binaries are built
   - ✅ Release is created
   - ✅ Homebrew formula is updated
   - ✅ Formula is pushed to homebrew-tap repository

### Verify Formula Update

1. Go to https://github.com/igusev/homebrew-tap/commits/main
2. You should see a new commit from GoReleaser:
   ```
   Brew formula update for glf version v0.2.1
   ```

3. Check the updated formula:
   ```bash
   # Update your tap
   brew update

   # Check the formula version
   brew info igusev/tap/glf
   ```

## Step 6: Update GLF Repository README

Add installation instructions to the main GLF README.md:

```markdown
## Installation

### Homebrew (macOS and Linux)

```bash
brew tap igusev/tap
brew install glf
```

### From Source

```bash
git clone https://github.com/igusev/glf.git
cd glf
make install
```
```

## Troubleshooting

### "Permission denied" when pushing formula

**Problem:** GoReleaser can't push to homebrew-tap repository

**Solution:**
- Verify token has `repo` scope
- Check token hasn't expired
- Ensure token is saved as `HOMEBREW_TAP_GITHUB_TOKEN` secret
- Verify release.yml uses the correct secret name

### Formula update fails

**Problem:** GoReleaser reports error when updating formula

**Solution:**
```bash
# Check .goreleaser.yml syntax
cd /path/to/glf
goreleaser check

# Test configuration
goreleaser release --snapshot --clean
```

### Formula not found after installation

**Problem:** `brew tap igusev/tap` succeeds but formula not found

**Solution:**
```bash
# Update Homebrew
brew update

# Verify tap was added
brew tap

# Check tap repository
ls $(brew --repository)/Library/Taps/igusev/homebrew-tap/Formula/
```

### Token security concerns

**Problem:** Worried about token security

**Solution:**
- Use fine-grained personal access tokens (beta) for better security
- Set minimal permissions (only `repo` scope)
- Set expiration date and rotate regularly
- Never commit token to repository
- Only store in GitHub Actions secrets

## Best Practices

1. **Never commit secrets** - Always use GitHub Actions secrets
2. **Test releases** - Use `goreleaser release --snapshot` to test locally
3. **Semantic versioning** - Follow semver for version tags (v1.2.3)
4. **Monitor workflows** - Check GitHub Actions after each release
5. **Keep formula simple** - Let GoReleaser handle updates
6. **Document changes** - Update CHANGELOG.md with each release

## What's Next?

After setup is complete:

1. ✅ Create releases with `git tag` and push
2. ✅ GoReleaser automatically updates the formula
3. ✅ Users can install with `brew install igusev/tap/glf`
4. ✅ Formula stays in sync with releases

## Repository URLs

- **Main Project:** https://github.com/igusev/glf
- **Homebrew Tap:** https://github.com/igusev/homebrew-tap
- **Releases:** https://github.com/igusev/glf/releases
- **Actions:** https://github.com/igusev/glf/actions

## Need Help?

- [GoReleaser Documentation](https://goreleaser.com/)
- [Homebrew Documentation](https://docs.brew.sh/)
- [GitHub Actions Documentation](https://docs.github.com/actions)
