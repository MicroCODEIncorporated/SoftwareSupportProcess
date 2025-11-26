# MicroCODE Software Support Process

**Subject:** Application Development and Maintenance Process
**Audience:** MicroCODE Customers
**Purpose:** Explanation of recommended App support process
**Author:** Tim McGuire
**Last Updated:** [To be updated]

## Background

Developing complex software applications and supporting time-critical operations requires a proactive and highly responsive software support process.

## The Development Circles Model

MicroCODE sees application development as a set of concentric **Circles**. Each Circle consists of:

- A code base used to build all parts of a solution
- A documentation set describing the use of the code base
- A team, or allocated time for personnel within a team, to support the Circle activities

> **Note:** Microsoft refers to these 'Circles' as 'Rings' - see [Managing Office 365 Updates](https://blogs.office.com/2015/08/12/managing-office-365-updates/)

### The Three Circles

The following application Circles must be supported concurrently:

#### 1. Internal Development Circle (ALPHA)

- **Git Branch:** `develop`
- **Purpose:** The 'next major release' currently being developed
- **Status:** Pre-production code, integration branch for new features
- **Activity:** Long-term development work
- **Testing:** Unit and Integration testing

#### 2. Plant Pilot Circle (BETA)

- **Git Branch:** `release/*` (long-lived branches)
- **Purpose:** The 'current major release' being tested and supported in selected customer sites
- **Status:** Pilot-ready 'Production Candidate'
- **Activity:** Multi-group testing before production promotion
- **Requirements:** Minimum of three (3) sites, varying environments, to exit Pilot phase
- **Testing:** System Acceptance Testing (SAT) and User Acceptance Testing (UAT)

#### 3. Plant Production Sites (PRODUCTION)

- **Git Branch:** `main`
- **Purpose:** The 'previous major release(s)' running in actual Production sites
- **Status:** Production-ready, stable releases
- **Activity:** Multiple versions may be in the field based on plant upgrade schedules
- **Support:** All versions must be supported concurrently until End of Life

## Git Workflow

We use this branching model that maps directly to our Circles:

### Branch Structure

```
main                (Production)
  ├── hotfix/*      (Emergency patches in main)
  ├── develop       (Alpha - Integration branch)
  ├── feature/*     (New features from main, accumulate in develop)
  │   └── bugfix/*  (Defect corrections in alpha test)
  └── release/*     (Beta - Forked from main, cherry-picks selected features)
      └── bugfix/*  (Defect corrections in beta test)
```

### Branch Types

#### `main` Branch

- **Circle:** [Production]()
- **Purpose:** Always contains production-ready code
- **Protection:** Protected, requires PR approval
- **Tagging:** Every release tagged with `vM.F.H` (e.g., `v2.11.322`)
- **Versioning:** [vM.F.H]()

#### `hotfix/*` Branches

- **Circle:** [Production]() → [Alpha]() → [Beta]()
- **Purpose:** Emergency patches for production issues
- **Naming:** `hotfix/{issue#}--{short-snake-name}`
- **Source:** Created from `main`
- **Merge:** Merged to `main` immediately, then merged to `develop` (and any open `release/*` branches)
- **Versioning:** [vM.F.H]()

#### `develop` Branch

- **Circle:** [Alpha]()
- **Purpose:** Integration branch for next release
- **Source:** All feature branches merge here
- **Status:** Pre-production, actively developed
- **Versioning:** [aM.F.H]()

#### `feature/*` Branches

- **Circle:** [Alpha]()
- **Purpose:** New functionality or improvements
- **Naming:** `feature/{issue#}--{short-snake-name}`
- **Source:** Created from `main`
- **Merge:** Merged to `develop` when complete for testing
- **Versioning:** [aM.F.H]()

#### `release/*` Branches

- **Circle:** [Beta]()
- **Purpose:** Long-lived branches for multi-group testing
- **Naming:** `release/b1.0.0` (semantic versioning)
- **Lifespan:** Remain open during extended testing periods
- **Merge:** Merged to `main` when testing complete
- **Versioning:** [bM.F.H]()

### Version Numbering (Semantic Versioning)

Alpha tags (aM.F.H) are on `develop` and Beta tags (bM.F.H) are on `release/*` branches; production tags (vM.F.H) are on `main`.

We use **Semantic Versioning (SemVer)**: `vM.F.H`

- **`M`** = Major Release - incremented on architecture, design, APIs or other major App changes
- **`F`** = Feature Release - incremental release of Feature(s) into a Major Release (.1. = 1st release of new features, .2. = 2nd, .3. = 3rd, etc.)
- **`H`** = Hotfix or Bugfix Patch - incremental fixes released to correct a particular vM.F, bM.F, or aM.F

**Examples:**

- `v2.1.0` - Major Version 2, 1st Feature Release, no hotfixes
- `a2.1.1` - Major Version 2, 1st Feature Release, 1st hotfix - ALPHA Build
- `b2.1.1` - Major Version 2, 1st Feature Release, 1st hotfix - BETA Build
- `v2.1.1` - Major Version 2, 1st Feature Release, 1st hotfix
- `v2.2.0` - Major Version 2, 2nd Feature Release (new features bundled), no hotfixes
- `v3.0.0` - Major Version 3 (architectural change), 1st Feature Release

See [MicroCODE Git Workflow](./DEVELOPER-GIT.md) - for detailed procedures and examples.

## Issue Tracking with GitHub Issues/Projects

We use **GitHub Issues** and **GitHub Projects** to track all software problems, enhancements, and requests. This replaces the previous SPR/SER tracking system.

### GitHub Issues

Every problem, request, or enhancement is tracked as a GitHub Issue:

- **Bug Reports:** Issues labeled with `bug` and severity labels
- **Enhancements:** Issues labeled with `enhancement`
- **Hotfixes:** Issues labeled with `hotfix` and linked to `hotfix/*` branches
- **Features:** Issues labeled with `feature` and linked to `feature/*` branches

### Issue Labels

We use a standardized label system:

**Type Labels:**

- `bug` - Software defect
- `enhancement` - New feature or improvement
- `hotfix` - Emergency production fix
- `documentation` - Documentation updates

**Severity Labels:**

- `severity-1` - System crash, complete loss of major component
- `severity-2` - Function fails, no workaround
- `severity-3` - Function fails, workaround available
- `severity-4` - Function fails, no production impact
- `severity-5` - Display/report/tool problem
- `severity-6` - Annoyance, requires business case

**Status Labels:**

- `alpha` - Work in Alpha Circle
- `beta` - Work in Beta Circle
- `production` - Work in Production Circle
- `blocked` - Waiting on dependency
- `ready-for-review` - Ready for PR review

### GitHub Projects

We use **GitHub Projects** (board view) to visualize workflow:

**Project Board Columns:**

1. **Backlog** - New issues, not yet prioritized
2. **Alpha (In Progress)** - Active development in `develop`
3. **Beta (Testing)** - Issues in `release/*` branches
4. **Production (Deployed)** - Issues merged to `main`
5. **Done** - Completed and verified

**Benefits:**

- Automatic linking between Issues, PRs, and commits
- Filter by labels, assignees, milestones
- Multiple views (board, table, roadmap)
- All tracking in one place (GitHub)

### Linking Issues to Code

When creating branches and PRs, link to Issues:

**Branch Naming with Issue Numbers:**

```
feature/0045--add-sidebar-builder  (GitHub Issue #45)
hotfix/0067--connection-retry      (GitHub Issue #67)
```

**PR Descriptions:**

```markdown
Fixes #67
Implements #45

Related to release v2.2.0
```

## Defect Severity and Response Times

| Severity | Description                                           | Impact                          | Response Time                     |
| :------: | :---------------------------------------------------- | :------------------------------ | :-------------------------------- |
|    1     | System crash, complete loss of major system component | Lost units or loss of integrity | Immediate (Hours/Days)            |
|    2     | Function fails, no work-around possible               | Lost units or loss of integrity | Immediate (Hours/Days)            |
|    3     | Function fails, work-around available                 | Loss of integrity               | Days/Weeks                        |
|    4     | Function fails, no production impact                  | Local support required          | Next Minor Release (Weeks/Months) |
|    5     | Display/Report/Tool problem                           | Possible future time loss       | Next Minor Release (Weeks/Months) |
|    6     | Annoyances                                            | Resource time loss              | Business Case required            |

## Testing Strategy

### Unit Testing

- **Location:** Alpha Circle (`feature/*` branches)
- **Responsibility:** Developers
- **Scope:** Application code level, standalone programs

### Integration Testing

- **Location:** Alpha Circle (`feature/*` branches)
- **Responsibility:** Devs
- **Scope:** System component interactions, functional requirements

### System Acceptance Testing (SAT)

- **Location:** Alpha Circle (`develop` branch)
- **Responsibility:** Devs and Testers
- **Scope:** Complete integrated software product validation

### User Acceptance Testing (UAT)

- **Location:** Beta Circle (`release/*` branches)
- **Responsibility:** Testers and Support Team
- **Scope:** Business requirements compliance, delivery acceptance

## Risk, Expense, and Urgency

The further out you go in the Circles, the greater the:

- **Risk** associated with defects
- **Expense** of defect correction
- **Urgency** of response required

Conversely, the further into the Circles you go, the greater the:

- **Time** available for changes
- **Flexibility** in implementation
- **Options** for solutions

## GitHub Best Practices

### Branch Protection

Protect these branches in GitHub:

- `main` - Require PR, require approvals, require status checks
- `develop` - Require PR, require status checks
- `release/*` - Require PR (optional, for long-lived branches)

### Pull Requests

- **Required for:** All merges to `main`, `develop`, and `release/*`
- **Title Format:** `[vM.F.H] Description` (e.g., `[v2.1.1] Fix connection retry`)
- **Description:** Link to related Issues, describe changes
- **Review:** At least one approval required for `main`

### Tags

- **Format:** `vM.F.H` (e.g., `v2.1.1`)
- **Production Tags:** `vM.F.H` tags are on `main` branch
- **Alpha Tags:** `aM.F.H` tags are on `develop` branch
- **Beta Tags:** `bM.F.H` tags are on `release/*` branches
- **Purpose:** Mark releases and builds in each Circle
- **Release Notes:** Use GitHub Releases feature to document changes

## Summary

This process combines:

1. **The Circles Model** - Your proven experience since 1983, representing parallel development activities
2. **GitFlow** - Industry-standard Git branching model
3. **GitHub Issues/Projects** - Modern, integrated tracking system
4. **Semantic Versioning** - Clear, predictable version numbering

The workflow ensures:

- Clean branch states at any moment
- Systematic naming conventions
- Protection of production code
- Dedicated channels for hotfixes
- Support for long-lived release testing
- All tracking in one integrated system (GitHub)

---

**For detailed Git commands and examples, see:** `DEVELOPER-GIT.md`
