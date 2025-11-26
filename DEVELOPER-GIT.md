# App Versions & MicroCODE Git Workflow

This guide mirrors the [MicroCODE Software Support Process](./DEVELOPER-SSP.md) (SSP) and documents how developers work with Git branches, semantic versioning, and the four GitFlow templates stored under `./.github/GIT - Issue Notes/`.

**`vM.F.H`**

- `M` = Major Release (architecture/design/API change)
- `F` = Feature Release (bundle of enhancements delivered together)
- `H` = Hotfix Patch (single production fix)

Tags use `vM.F.H` on `main`. Branch names never embed the version except for the long-lived `release/bM.F.0` branch.

<p align="left"><img src="./.images/mcode-git-workflow-dark.png" width="1024" title="MicroCODE Git Workflow" style="border: 0.5px solid lightgray;"></p>

## 1. Branch Types & When to Use Them

**Key principle:** Releases are completely decoupled from `develop`.
You select specific, tested features from their individual branches and combine them on a safe foundation (`main`).

This keeps Production stable, Beta Release branches focused and testable, and `develop` free to accumulate features without blocking releases.

```
main                (Production)
  ├── hotfix/*      (Emergency patches in main)
  ├── develop       (Alpha - Integration branch)
  ├── feature/*     (New features from main, accumulate in develop)
  │   └── bugfix/*  (Defect corrections in alpha test)
  └── release/*     (Beta - Forked from main, cherry-picks selected features)
      └── bugfix/*  (Defect corrections in beta test)
```

| Branch                           | Purpose                                                | Source                                                                                  | Merge Target(s)                              | Documenation Template                              |
| -------------------------------- | ------------------------------------------------------ | --------------------------------------------------------------------------------------- | -------------------------------------------- | -------------------------------------------------- |
| `main`                           | Production code only. Always tagged. Protected branch. | n/a                                                                                     | n/a                                          | --                                                 |
| `hotfix/{issue#}--{short-name}`  | Emergency production fix.                              | `main`                                                                                  | `main`, `develop`, open `release/*`          | `GIT - Issue Notes/hotfix/ HOTFIX - template.md`   |
| `develop`                        | Alpha integration branch where features accumulate.    | `main`                                                                                  | None: integration and testing branch         | --                                                 |
| `release/bM.F.0`                 | Beta testing branch. Holds selected features for RC.   | `main` + cherry-picks directly from `feature/*`                                         | `main`, `develop`                            | `GIT - Issue Notes/release/ RELEASE - template.md` |
| `feature/{issue#}--{short-name}` | Implements a single GitHub Issue (new capability).     | `main`                                                                                  | `develop` and cherry-picked into `release/*` | `GIT - Issue Notes/feature/ FEATURE - template.md` |
| `bugfix/{issue#}--{short-name}`  | Fixes discovered while in Alpha or Beta testing.       | `feature/*` (canonical source) or `release/bM.F.0` (if feature branch no longer exists) | `feature/*`, `develop`, or `release/*`       | `GIT - Issue Notes/bugfix/ BUGFIX - template.md`   |

Notes:

- For Beta bugfixes, we create from the affected `feature/*` branch (canonical source) to ensure fixes persist even if the `release/*` branch is abandoned.

Naming rules:

- `nnnn` = the GitHub Issue number zero-padded to four digits.
- Use double dashes between the issue number and the short snake-case summary, e.g. `feature/0045--add-sidebar-builder`.
- Only `release/bM.F.0` branch includes version numbers; all other branches reference Issues.
- The `develop` branch will be tagged with alpha semantic version for testers, `aM.F.H`
- When a `release` branch is promoted to `main` its Hotfix Patch number is reset to `0`.

---

## 2. Version Bumping Rules & Commands

### Major Release → `v(M+1).0.0`

The only difference between a MAJOR Release and a FEATURE Release is the nature of the Features being included.
Use when architecture, APIs, or significant design shifts occur, this is a MAJOR Release.

MAJOR Release test questions:

1. Does the Features set break existing Customer extensions or API usage? Yes = MAJOR
2. Does the Feature set force a change in Customer connected services or suppliers (Stripe, Mailgun, etc.)? Yes = MAJOR

### Feature Release → `vM.(F+1).0`

**Critical:** Release branches are forked from `main` (proven safe code) and we cherry-pick **directly from `feature/*` branches**, NOT from `develop`. This completely decouples releases from `develop`, preventing "release paralysis" and giving you full control over what ships.

**Why NOT cherry-pick from `develop`?**

- `develop` may have incomplete features, experimental code, or integration issues
- Cherry-picking from `develop` still ties you to its unstable state
- You want to select specific, tested features—not whatever happens to be in `develop`

**Planning a Release:**

1. Review completed `feature/*` branches (regardless of `develop` state)
2. Select which features are most needed by customers
3. Fork `release/bM.F.0` from `main` (safe starting point)
4. Cherry-pick **directly from the `feature/*` branch commits**
5. Test a focused, manageable integration of `main` + `features`

```bash
# Develop the major feature
git checkout main
git checkout -b feature/0100--new-render-core
git checkout -b feature/0144--new-json-templates
git checkout -b feature/0151--new-api-v2
# develop, test, document with FEATURE template

# implement feature(s) and merge into develop for alpha testing
git checkout develop
git merge feature/0100--new-render-core
git merge feature/0144--new-json-templates
git merge feature/0151--new-api-v2
git tag a2.0.0

# Fork a new release from main (safe), cherry-pick directly from feature branch
git checkout main
git checkout -b release/b2.0.0
git cherry-pick feature/0100--new-render-core
git cherry-pick feature/0144--new-json-templates
# 🚫 intentionally skip feature/0151--new-api-v2 (it stays in develop until it’s ready)
git tag b2.0.0

# run SAT/UAT, when 100%..
git checkout main
git merge release/b2.0.0
git tag v2.0.0
```

### Hotfix (Production) → `vM.F.(H+1)`

Used when production needs an immediate correction. The hotfix increments the `H` component of SemVer.

```bash
git checkout main
git checkout -b hotfix/0067--connection-retry

# fix, test, document with HOTFIX template
git checkout main
git merge hotfix/0067--connection-retry

# increment H
git tag v2.1.(H+1)

# echo into develop branch to keep it current
git checkout develop
git merge hotfix/0067--connection-retry

# increment H, may *not* be the same as main vM.F.(H)
git tag a2.1.(H+1)

# echo into any open release branch as well
git checkout release/b2.2.0
git merge hotfix/0067--connection-retry

# increment H, may *not* be the same as main vM.F.(H), or alpha aM.F.(H)
# NOTE: This does *not* change the name of the release/* branch.
git tag b2.2.(H+1)
```

### Bugfix (Beta) → made to canonical source (feature/\*)

Use when a defect is found while the Beta Release Candidate (RC) is in SAT/UAT.
The bugfix is made to the original feature/\* branch, then echoed into release/\*, and develop.
The result is tagged `bM.F.(H+1)` in the release/\* branch, the branch name is unchanged.

```bash
git checkout feature/0100--new-render-core
git checkout -b bugfix/0067--connection-retry

# fix, test, document with BUGFIX template
git checkout feature/0100--new-render-core
git merge bugfix/0067--connection-retry

# echo into develop branch to keep it current
git checkout develop
git merge bugfix/0067--connection-retry

# increment H, may *not* be the same as main vM.F.(H)
git tag a2.1.(H+1)

# echo into any open release branch as well
git checkout release/b2.2.0
git merge bugfix/0067--connection-retry

# increment H, may *not* be the same as main vM.F.(H), or alpha aM.F.(H)
git tag b2.2.(H+1)
```

## 3. Branch Naming Quick Reference

Patterns:

```
main
hotfix/{issue#}--{short-snake-name}
develop
feature/{issue#}--{short-snake-name}
bugfix/{issue#}--{short-snake-name}
release/bM.F.0
```

Examples:

```
main
hotfix/0067--connection-retry
develop
feature/0045--add-sidebar-builder
bugfix/0080--toast-null-check
release/b2.2.0
```

## 4. Practical Checklist

- **PR Titles:** `[vM.F.H] Description` (e.g.: `[v2.2.0] Add sidebar builder`)
- **Issue Linking:** Include `Fixes #NNNN` or `Closes #NNNN` in every PR description.
- **Branch Protection:** `main`, `develop`, and active `release/*` require PR approval + status checks.
- **Tags:** Always tag releases on `main`. Hotfix tags use the next `H` value.
- **Documentation:** Use the Markdown templates in `./.github/GIT - Issue Notes/` to capture context before merging.
- **Auto-delete:** Enable automatic branch deletion in GitHub settings for merged branches.

## 5. Template Index

- Feature work: `./.github/GIT - Issue Notes/feature/FEATURE - template.md`
- Bugfix during Beta: `./.github/GIT - Issue Notes/bugfix/BUGFIX - template.md`
- Hotfix to Production: `./.github/GIT - Issue Notes/hotfix/HOTFIX - template.md`
- Release coordination: `./.github/GIT - Issue Notes/release/RELEASE - template.md`

Each template aligns with the MicroCODE Software Support Process (SSP) expectations
([Problem → Observation → Resolution → Verification]()) and should be completed before requesting a PR review.

- See [DEVELOPER-SSP.md](./DEVELOPER-SSP.md) for more information.
