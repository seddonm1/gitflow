# Gitflow GitHub Workflows

This directory contains generic GitHub workflows that implement the gitflow branching model with automated locking/unlocking of the develop branch.

## Directory Structure

```
.github/workflows/
├── core/                           # Generic gitflow workflows (don't modify)
│   ├── create_release.yml          # Creates release branches and PRs
│   ├── create_hotfix.yml           # Creates hotfix branches and PRs
│   ├── main_merge_handler.yml      # Handles merges to main (tagging, back-merge)
│   ├── pr_validation.yml          # Validates PRs to develop (locking mechanism)
│   └── main_pr_validation.yml      # Validates PRs to main (release/hotfix only)
└── user/                           # Project-specific implementations
    ├── version_bump.yml            # YOUR implementation (required)
    ├── develop_ci.yml              # YOUR CI for develop PRs (optional)
    ├── release_ci.yml              # YOUR CI for release/hotfix PRs (optional)
    └── examples/                   # Language-specific examples
        └── python_version_bump.yml
```

## Setup Instructions

### 1. Implement Version Bumping (Required)

```bash
# Copy the Python example and customize for your project
cp .github/workflows/user/examples/python_version_bump.yml .github/workflows/user/version_bump.yml
```

This workflow handles version bumping for both your changelog and project files. It uses [Keep a Changelog](https://keepachangelog.com/) format and verifies both systems produce the same version number.

### 2. Create CHANGELOG.md (Required)

Create a `CHANGELOG.md` file in your repository root following the [Keep a Changelog](https://keepachangelog.com/) format:

```markdown
# Changelog

## [Unreleased]

### Added
- Your new features here

### Changed
- Your modifications here

### Fixed
- Your bug fixes here

## [1.0.0] - 2024-01-01

### Added
- Initial release
```

### 3. Implement CI Workflows (Optional)

**Develop CI** (runs on PRs to develop):
```bash
# Create develop_ci.yml for PR validation
cp .github/workflows/user/develop_ci.yml.example .github/workflows/user/develop_ci.yml
```

**Release CI** (runs on release/hotfix PRs to main):
```bash
# Create release_ci.yml for release validation
cp .github/workflows/user/release_ci.yml.example .github/workflows/user/release_ci.yml
```

## How It Works

### Branch Locking Mechanism

The develop branch is "locked" using a **PR validation approach**:

1. **Lock**: When `release/*` or `hotfix/*` branches exist, PRs to develop are automatically blocked
2. **Unlock**: When release/hotfix is merged to main and the source branch is deleted

### Core Workflows

#### 1. Create Release (`create_release.yml`)
- **Trigger**: Manual dispatch from `develop` branch
- **Options**: `major` or `minor` version bump (no patch - use hotfix for patches)
- **Actions**:
  - Validates conditions (must be from develop, no active release/hotfix)
  - Calls your `version_bump.yml` with the selected bump type
  - Creates `release/X.Y.Z` branch with committed version changes
  - Opens PR to `main`
  - Develop branch is now "locked" by PR validation

#### 2. Create Hotfix (`create_hotfix.yml`)
- **Trigger**: Manual dispatch from `main` branch
- **Version**: Always `patch` bump (automatic)
- **Input**: Description of the hotfix
- **Actions**:
  - Validates conditions (must be from main, no active release/hotfix)
  - Calls your `version_bump.yml` with `patch`
  - Creates `hotfix/X.Y.Z` branch with committed version changes
  - Opens PR to `main`
  - Develop branch is now "locked" by PR validation

#### 3. PR Validation (`pr_validation.yml`)
- **Trigger**: Any PR to `develop`
- **Actions**:
  - Blocks PRs if `release/*` or `hotfix/*` branches exist (with helpful message)
  - Validates PRs come from `feature/*` branches
  - Verifies `CHANGELOG.md` exists
  - Calls your `develop_ci.yml` if validation passes

#### 4. Main PR Validation (`main_pr_validation.yml`)
- **Trigger**: Any PR to `main`
- **Actions**:
  - Blocks PRs unless they're from `release/*` or `hotfix/*` branches
  - Explains users must use "Create Release" or "Create Hotfix" workflows
  - Verifies `CHANGELOG.md` exists and contains the specific version entry
  - Calls your `release_ci.yml` for valid PRs

#### 5. Main Merge Handler (`main_merge_handler.yml`)
- **Trigger**: Merge to `main` branch
- **Actions**:
  - Detects if merge was from `release/*` or `hotfix/*`
  - Creates version tag (e.g., `v1.2.3`)
  - Extracts changelog content for the version
  - Creates GitHub release with changelog content as release notes
  - Merges changes back to `develop`
  - Deletes the source branch
  - Develop branch is now "unlocked" (no blocking branches exist)

## Usage

### Creating a Release

1. **Update changelog**: Add your changes to the `[Unreleased]` section of `CHANGELOG.md`
2. Ensure you're on the `develop` branch
3. Go to Actions → "Create Release"
4. Choose `major` or `minor`
5. The workflow will:
   - Convert `[Unreleased]` section to versioned release in changelog
   - Bump version in project files (and verify they match)
   - Create `release/X.Y.Z` branch with both changes committed
   - Open PR to main with release notes
   - Lock develop branch (feature PRs blocked until release completes)
   - Run release CI on the PR

### Creating a Hotfix

1. **Update changelog**: Add your hotfix changes to the `[Unreleased]` section of `CHANGELOG.md`
2. Ensure you're on the `main` branch
3. Go to Actions → "Create Hotfix"
4. Provide description of the fix
5. The workflow will:
   - Convert `[Unreleased]` section to patch version in changelog
   - Bump patch version in project files (and verify they match)
   - Create `hotfix/X.Y.Z` branch with both changes committed
   - Open PR to main with hotfix notes
   - Lock develop branch (feature PRs blocked until hotfix completes)
   - Run release CI on the PR

### Feature Development

1. Create feature branches from `develop`: `feature/my-feature-name`
2. Open PRs to `develop`
3. If develop is locked, the PR will be blocked with explanation
4. Once validation passes, develop CI runs automatically
5. After review/approval, merge to develop

### Release/Hotfix PRs

- PRs to main are **only allowed** from `release/*` or `hotfix/*` branches
- Manual PRs to main are blocked with instructions to use workflows
- Release CI runs automatically on valid PRs
- After merge, changes are automatically back-merged to develop

## CI Integration

### Develop CI Flow
```
PR to develop → Validation → develop_ci.yml → Review → Merge
```

### Release CI Flow
```
Release/Hotfix PR → Validation → release_ci.yml → Review → Merge → Tag & GitHub Release & Back-merge
```

## Changelog Integration

This gitflow implementation is deeply integrated with [Keep a Changelog](https://keepachangelog.com/):

### Workflow
1. **Development**: Add changes to `[Unreleased]` section as you work
2. **Release**: Automation converts `[Unreleased]` → `[1.2.3]` and bumps project version
3. **GitHub Release**: Uses actual changelog content as release notes

### Version Verification
The system ensures changelog and project versions stay in sync:
- Both changelog and project files are bumped independently
- Verification step ensures they calculate the same version
- Process fails if versions don't match (prevents drift)

## Branch Protection Recommendations

### Main Branch Protection
- Require PR reviews (1+ approvers)
- Dismiss stale reviews when new commits pushed
- Require status checks to pass:
  - "Validate PR to Main"
  - "Run Release CI"
- Restrict pushes (no direct commits)
- Require branches to be up to date

### Develop Branch Protection
- Require PR reviews (1+ approvers)
- Require status checks to pass:
  - "Check Develop Branch Lock Status"
  - "Run CI Tests"
- Restrict pushes (no direct commits)
- Require branches to be up to date

## Troubleshooting

### "Develop branch is locked"
- Check for active `release/*` or `hotfix/*` branches
- Wait for them to be merged to main and automatically deleted
- If stuck, manually delete the blocking branch

### "Invalid PR to Main"
- Don't create manual PRs to main
- Use "Create Release" workflow from develop
- Use "Create Hotfix" workflow from main

### Version bump fails
- Check your `version_bump.yml` implementation
- Ensure the version bump command works for your project
- Verify the commit step is present (changes are lost without it)

### CI not running
- Check that `develop_ci.yml` and `release_ci.yml` exist if referenced
- Ensure workflow files are valid YAML
- Check GitHub Actions permissions

### Permission errors
- Ensure `GITHUB_TOKEN` has sufficient permissions
- For some operations, you may need a Personal Access Token (PAT)
