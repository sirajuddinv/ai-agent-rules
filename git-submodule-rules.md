---
title: Git Submodule Management
description: Protocols for managing submodules,
    preventing detached HEAD states, proper branch tracking, and
    descriptive parent
    commit messages.
category: Git & Repository Management
---


# Git Submodule Management Rules

This document outlines the strict protocols for managing Git submodules to prevent "detached HEAD" states, lost
commits, and synchronization issues.

***

## 1. Core Principle: Branch Tracking

- **Never Commit to Detached HEAD**: Submodules often check out a specific commit SHA by default, placing them in a

    "detached HEAD" state.

- **Mandatory Checkout**: Before making ANY changes to a submodule, you MUST explicitly check out a branch (usually

    `master`or`main`).

- Command: `cd <submodule-dir> && git checkout <branch-name>`

- **Verification**: Run `git branch --show-current` inside the submodule to confirm you are on a branch before creating

    files or committing.

***

### 2. Workflow for Updating Submodules

When you need to add files, update rules, or modify content within a submodule:

1. **Enter & Checkout**:

    ```bash
    cd <submodule-path>
    git checkout <default-branch> # master or main
    git pull origin <default-branch> # Ensure you are up to date
    ```

1. **Arranged Commit Construction**:

- Follow the [git-atomic-commit-construction-rules.md](./git-atomic-commit-construction-rules.md) to group and stage

    related changes logically.

1. **Commit in Submodule**:

    ```bash
    git add .
    git commit -m "feat(scope): description..."
    ```

1. **Push Submodule (Optional but Recommended)**:

- If you have write access, push the submodule changes immediately to avoid them existing only locally.

- `git push origin <branch-name>`

1. **Update Parent Repository**:

- Return to the parent root: `cd ..`

- Stage the submodule folder: `git add <submodule-path>`

- Commit the pointer update (describe what changed, not just that it updated):

    ```bash
    git commit -m "chore(submodule): update <submodule-name> with <change-summary>

    - Describe specific changes made in the submodule
    - Follow standard commit message rules"
    ```

***

### 3. Recovery from Detached HEAD

If you accidentally commit to a detached HEAD:

1. **Identify the Commit**: Run `git log -1`or`git reflog` to get the SHA of your new commit.

1. **Checkout Branch**: `git checkout <branch-name>` (master or main)

1. **Cherry-Pick**: `git cherry-pick <commit-sha>`

1. **Push & Sync**: Push the branch and update the parent repository as described above.

***

### 4. Automation Safeguards

- **AI Tooling**: When an AI agent is tasked with modifying a submodule, it must first execute
  `git checkout <default-branch>` as a prerequisite step.

- **Status Check**: The agent should verify `git status` output does not contain "HEAD detached" before proceeding
  with edits.

***

### 5. Migrating a Tracked Directory to a Submodule

When replacing a directly-tracked directory (e.g., `pevers/` containing LFS-tracked archives) with a proper Git
submodule, all related changes MUST be coupled in a **single atomic commit**.

#### Workflow:

1. **Remove old tracked files from the index** (keep local copies if needed for reference):

    ```bash
    git rm --cached <old-dir>/<file>    # Remove specific files
    # OR
    git rm -r --cached <old-dir>/       # Remove entire directory from index
    ```

2. **Add the submodule** (if not already added via `git submodule add`):

    ```bash
    git add .gitmodules <new-submodule-path>
    ```

3. **Update `.gitignore`** — Remove stale ignore/negation patterns that referenced the old directory. Submodules
   manage their own `.gitignore` internally, so the parent repository does NOT need ignore rules for the submodule
   path.

    ```bash
    # Remove lines like:
    #   pevers/*
    #   !pevers/*.zip
    #   !pevers/*.7z
    git add .gitignore
    ```

4. **Stage and commit atomically** — All four changes (file removal, `.gitmodules`, submodule pointer, `.gitignore`
   cleanup) MUST be in the same commit:

    ```bash
    git commit -m "refactor(<scope>): migrate <old-dir> to <new-dir> git submodule

    Replace the directly-tracked <old-dir>/ directory with a proper
    <new-dir> git submodule pointing to <remote-url> at <commit-sha>.

    Remove stale <old-dir>/* ignore/negation patterns from .gitignore --
    the submodule manages its own repository and does not need parent
    .gitignore rules."
    ```

#### Why Atomic:

- **`.gitignore` patterns referencing the old directory** become stale after migration. If left in a separate commit,
  there is an intermediate state where the patterns reference a non-existent directory.
- **The submodule pointer and `.gitmodules`** are functionally coupled — one without the other is a broken state.
- **The old tracked files** must be removed in the same commit to avoid a state where both the old files and the
  submodule coexist.

#### LFS Consideration:

If the old directory contained LFS-tracked files (`.zip`, `.7z`), ensure `git lfs install` has been run in the
parent repository before committing. The submodule will manage its own LFS tracking independently.
