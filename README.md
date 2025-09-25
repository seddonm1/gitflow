# Gitflow Workflow Rules

This document outlines the branching model and rules for our development process, incorporating Pull Requests, automation, and a code-freeze mechanism for the `develop` branch during release cycles.

### Core Principles

*   **`main`**: This is the production branch. Its history consists only of versioned releases. It is always stable.
*   **`develop`**: This is the main integration branch for all new features. It enters a **locked** state (code freeze) during release or hotfix procedures to ensure stability.
*   **Versioning**: All version tags on `main` **must** follow **Semantic Versioning (SemVer)** in the format `MAJOR.MINOR.PATCH`.

---

## Workflow Rules

#### 1. Feature Branches (`feature/*`)
*   **Rule 1.1:** Must be created from the `develop` branch.
*   **Rule 1.2:** To merge a completed feature, a Pull Request (PR) must be opened from the `feature` branch to the `develop` branch.
*   **Rule 1.3:** This PR can only be merged when the `develop` branch is in an **unlocked** state. If `develop` is locked, the feature PR must wait.
*   **Rule 1.4:** Once the PR is reviewed, approved, and merged, the feature branch should be deleted.

#### 2. Release Branches (`release/*`)
*   **Rule 2.1:** Must be created from the `develop` branch.
*   **Rule 2.2 (`Triggers Lock`):** The creation of a `release` branch immediately puts the `develop` branch into a **locked** state. No new features can be merged into `develop`.
*   **Rule 2.3:** A Pull Request is opened from the `release` branch to `main`. Only release-critical bug fixes should be committed to the `release` branch.
*   **Rule 2.4:** When the release is finalized and the PR is merged into `main`, the commit on `main` must be tagged with a new version number (e.g., `1.0.0`).

#### 3. Hotfix Branches (`hotfix/*`)
*   **Rule 3.1:** Must be created from the `main` branch (from the specific version tag that needs fixing).
*   **Rule 3.2 (`Triggers Lock`):** The creation of a `hotfix` branch immediately puts the `develop` branch into a **locked** state.
*   **Rule 3.3:** A Pull Request is opened from the `hotfix` branch to `main`.
*   **Rule 3.4:** When the fix is complete and the PR is merged into `main`, the commit on `main` must be tagged with a new patch version number (e.g., `1.0.1`).

#### 4. The `main` Branch
*   **Rule 4.1:** This branch is protected and only accepts changes through approved Pull Requests from `release` and `hotfix` branches.
*   **Rule 4.2 (`Triggers Unlock`):** Every successful merge into `main` must trigger an **automated process** that merges the source branch (`release` or `hotfix`) back into `develop`. This automated back-merge is the event that **unlocks** the `develop` branch.

#### 5. The `develop` Branch
*   **Rule 5.1:** This branch can be in one of two states:
    *   **Unlocked (Default):** Can accept Pull Requests from `feature` branches.
    *   **Locked (Code Freeze):** Cannot accept merges from `feature` branches.
*   **Rule 5.2:** It becomes **locked** the moment a `release/*` or `hotfix/*` branch is created.
*   **Rule 5.3:** It becomes **unlocked** only after the automated back-merge from a `release` or `hotfix` branch is successfully completed.

---

## Locking Lifecycle Summary

1.  **Lock `develop`**: A developer creates a `release/*` or `hotfix/*` branch. The `develop` branch is now locked, preventing new features from being merged.

2.  **Work in Isolation**: Feature development can continue in `feature` branches, but their PRs targeting `develop` will be paused. The release/hotfix work is completed on its own branch.

3.  **Merge to `main`**: The `release`/`hotfix` PR is approved, merged into `main`, and tagged with a new version.

4.  **Unlock `develop`**: An automated action merges the changes from the `release`/`hotfix` branch back into `develop` and removes the lock. Normal feature merging can now resume.
