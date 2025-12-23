# Git Branching & Versioning Strategy

**Project**: Church Data Analytics Platform
**Version Control**: Git + GitHub
**Date**: 2025-12-23

---

## Current Branch Structure

### Branch Overview
```
main (production-ready code)
├── develop (integration branch)
├── feature/* (new features)
├── bugfix/* (bug fixes)
├── hotfix/* (urgent production fixes)
├── release/* (release preparation)
└── claude/* (AI assistant work branches)
```

---

## Branching Strategy: GitFlow (Modified)

### Why GitFlow?
- ✅ **Clear separation** of production, development, and feature work
- ✅ **Parallel development** - multiple features can be developed simultaneously
- ✅ **Release management** - structured release preparation
- ✅ **Hotfix support** - urgent fixes without disrupting development
- ✅ **Rollback capability** - tagged releases for easy rollback

### Branch Types & Purposes

---

## 1. Main Branches (Long-lived)

### `main` Branch
**Purpose**: Production-ready code only

**Rules**:
- ✅ **Always stable** - every commit should be deployable
- ✅ **Protected** - require pull request reviews
- ✅ **Tagged** - every merge gets a version tag (v1.0.0, v1.1.0, etc.)
- ✅ **Deployed automatically** - triggers production deployment
- ❌ **No direct commits** - only merge from `release/*` or `hotfix/*`

**Merge Sources**:
- `release/*` branches (for planned releases)
- `hotfix/*` branches (for emergency fixes)

**Example Commands**:
```bash
# Never commit directly to main
git checkout main  # ❌ DON'T DO THIS for development

# Only merge from release or hotfix
git checkout main
git merge --no-ff release/v1.2.0
git tag -a v1.2.0 -m "Release version 1.2.0"
git push origin main --tags
```

---

### `develop` Branch
**Purpose**: Integration branch for features (pre-production)

**Rules**:
- ✅ **Latest development code** - all features merge here first
- ✅ **Should be stable** but may have minor bugs
- ✅ **Continuous integration** - runs tests on every commit
- ✅ **Staging environment** - auto-deploys to staging server
- ❌ **Not production-ready** - needs testing before release

**Merge Sources**:
- `feature/*` branches (completed features)
- `bugfix/*` branches (non-critical bug fixes)

**Example Commands**:
```bash
# Create develop branch (one-time setup)
git checkout -b develop main
git push -u origin develop

# Merge feature into develop
git checkout develop
git merge --no-ff feature/church-directory
git push origin develop
```

---

## 2. Supporting Branches (Short-lived)

### `feature/*` Branches
**Purpose**: Develop new features

**Naming Convention**:
```
feature/church-directory
feature/rss-ingestion
feature/whisper-transcription
feature/bible-reference-extraction
feature/theme-classification
feature/map-visualization
```

**Lifecycle**:
```bash
# Start new feature
git checkout -b feature/church-directory develop

# Work on feature (commit frequently)
git add .
git commit -m "Add church directory page with map"
git push -u origin feature/church-directory

# Merge back to develop when complete
git checkout develop
git merge --no-ff feature/church-directory
git push origin develop

# Delete feature branch
git branch -d feature/church-directory
git push origin --delete feature/church-directory
```

**Rules**:
- ✅ Branch from: `develop`
- ✅ Merge back to: `develop`
- ✅ Naming: `feature/<description>`
- ✅ Lifespan: Days to weeks
- ✅ Delete after merge

**Pull Request Template** (for feature branches):
```markdown
## Feature: Church Directory

### Description
Implements the church directory page with interactive map showing all churches.

### Related User Stories
- Story 1.1: Browse Church Directory
- Story 1.3: View Church Details

### Changes
- Added `/app/churches/page.tsx` with church list and map
- Integrated Mapbox for interactive map
- Added church search and filter components
- Created Supabase query for church data

### Testing
- [ ] Manual testing on dev environment
- [ ] Tested on mobile devices
- [ ] Accessibility checks (keyboard navigation, screen reader)
- [ ] Cross-browser testing (Chrome, Firefox, Safari)

### Screenshots
[Attach screenshots]

### Checklist
- [ ] Code follows project style guidelines
- [ ] Self-review completed
- [ ] Comments added for complex logic
- [ ] Tests added/updated (if applicable)
- [ ] Documentation updated (if needed)
- [ ] No console errors or warnings
```

---

### `bugfix/*` Branches
**Purpose**: Fix bugs discovered in develop branch

**Naming Convention**:
```
bugfix/sermon-audio-player-crash
bugfix/church-map-loading-issue
bugfix/search-results-duplicate
```

**Lifecycle**:
```bash
# Start bugfix
git checkout -b bugfix/sermon-audio-player-crash develop

# Fix the bug
git add .
git commit -m "Fix audio player crash when sermon has no audio file"
git push -u origin bugfix/sermon-audio-player-crash

# Merge back to develop
git checkout develop
git merge --no-ff bugfix/sermon-audio-player-crash
git push origin develop
git branch -d bugfix/sermon-audio-player-crash
```

**Rules**:
- ✅ Branch from: `develop`
- ✅ Merge back to: `develop`
- ✅ Naming: `bugfix/<issue-description>`
- ✅ Lifespan: Hours to days
- ✅ Delete after merge

---

### `hotfix/*` Branches
**Purpose**: Urgent fixes for production issues

**Naming Convention**:
```
hotfix/v1.0.1-sermon-player-security
hotfix/v1.1.1-database-connection
```

**Lifecycle**:
```bash
# Start hotfix from main (production)
git checkout -b hotfix/v1.0.1-sermon-player-security main

# Fix the critical issue
git add .
git commit -m "Fix XSS vulnerability in sermon player"

# Merge to main AND develop
git checkout main
git merge --no-ff hotfix/v1.0.1-sermon-player-security
git tag -a v1.0.1 -m "Hotfix: Sermon player security"
git push origin main --tags

git checkout develop
git merge --no-ff hotfix/v1.0.1-sermon-player-security
git push origin develop

# Delete hotfix branch
git branch -d hotfix/v1.0.1-sermon-player-security
```

**Rules**:
- ✅ Branch from: `main` (production)
- ✅ Merge back to: `main` AND `develop`
- ✅ Naming: `hotfix/v<version>-<description>`
- ✅ Lifespan: Hours
- ✅ Tag main with new version
- ⚠️ **URGENT ONLY** - for critical production bugs

---

### `release/*` Branches
**Purpose**: Prepare for production release

**Naming Convention**:
```
release/v1.0.0
release/v1.1.0
release/v2.0.0
```

**Lifecycle**:
```bash
# Start release from develop
git checkout -b release/v1.1.0 develop

# Finalize release (bump versions, update changelog)
# Edit package.json: "version": "1.1.0"
# Update CHANGELOG.md with release notes
git add package.json CHANGELOG.md
git commit -m "Bump version to 1.1.0"

# Merge to main
git checkout main
git merge --no-ff release/v1.1.0
git tag -a v1.1.0 -m "Release version 1.1.0"
git push origin main --tags

# Merge back to develop (important!)
git checkout develop
git merge --no-ff release/v1.1.0
git push origin develop

# Delete release branch
git branch -d release/v1.1.0
```

**Rules**:
- ✅ Branch from: `develop`
- ✅ Merge to: `main` AND `develop`
- ✅ Naming: `release/v<version>`
- ✅ Lifespan: Days
- ✅ Only bug fixes, version bumps, docs
- ❌ No new features

**Release Checklist**:
```markdown
## Release v1.1.0 Checklist

### Pre-Release
- [ ] All features merged to develop
- [ ] All tests passing on develop
- [ ] Manual QA completed on staging
- [ ] Version bumped in package.json
- [ ] CHANGELOG.md updated
- [ ] Breaking changes documented
- [ ] Migration scripts ready (if needed)

### Release
- [ ] Create release branch
- [ ] Final testing on release branch
- [ ] Merge to main
- [ ] Tag version on main
- [ ] Deploy to production
- [ ] Monitor production for issues (first 24h)

### Post-Release
- [ ] Merge release back to develop
- [ ] Create GitHub release with notes
- [ ] Announce release (email, Slack, etc.)
- [ ] Update project board
- [ ] Close milestone
```

---

### `claude/*` Branches
**Purpose**: Work done by Claude Code (AI assistant)

**Naming Convention**:
```
claude/create-readme-review-6TwlW
claude/implement-church-directory-8KxPq
claude/fix-audio-player-2NmLk
```

**Rules**:
- ✅ Branch from: Current branch (usually `develop` or feature branch)
- ✅ Naming: `claude/<task-description>-<session-id>`
- ✅ Session ID ensures uniqueness
- ✅ Converted to regular branches via PR
- ✅ Reviewed by human before merge

**Lifecycle**:
```bash
# Claude creates branch automatically
# Example: claude/create-readme-review-6TwlW

# Human reviews the work
# Option 1: Merge directly if approved
git checkout develop
git merge --no-ff claude/create-readme-review-6TwlW
git push origin develop

# Option 2: Create PR for team review
gh pr create \
  --base develop \
  --head claude/create-readme-review-6TwlW \
  --title "Add comprehensive README and documentation" \
  --body "Claude created documentation for project vision and architecture"
```

**PR Review Process**:
1. Claude pushes to `claude/*` branch
2. Human reviews changes
3. If approved → merge to target branch (develop/feature)
4. If changes needed → provide feedback, Claude updates
5. Delete `claude/*` branch after merge

---

## 3. Versioning Strategy

### Semantic Versioning (SemVer)

**Format**: `MAJOR.MINOR.PATCH` (e.g., v1.2.3)

```
v1.0.0
 │ │ │
 │ │ └─── PATCH: Bug fixes (backward compatible)
 │ └───── MINOR: New features (backward compatible)
 └─────── MAJOR: Breaking changes (NOT backward compatible)
```

**Examples**:
```
v0.1.0  → MVP development (pre-release)
v0.2.0  → Added transcription feature
v1.0.0  → First production release (MVP complete)
v1.1.0  → Added Bible reference extraction
v1.1.1  → Fixed audio player bug
v1.2.0  → Added theme classification
v2.0.0  → Breaking API changes
```

### Version Numbering Rules

#### MAJOR (v1.0.0 → v2.0.0)
**When to increment**:
- Breaking API changes
- Database schema changes requiring migration
- Removing features
- Major architecture changes

**Examples**:
- Changed Supabase API endpoints
- Renamed database tables
- Removed support for RSS format

#### MINOR (v1.0.0 → v1.1.0)
**When to increment**:
- New features added
- New API endpoints
- Performance improvements
- Non-breaking enhancements

**Examples**:
- Added transcription feature
- New visualization dashboard
- Theme classification
- User accounts

#### PATCH (v1.0.0 → v1.0.1)
**When to increment**:
- Bug fixes
- Security patches
- Minor UI tweaks
- Performance optimizations (no new features)

**Examples**:
- Fixed sermon player crash
- Security vulnerability patch
- UI button alignment
- Database query optimization

### Pre-release Versions

**Alpha**: `v1.0.0-alpha.1` (internal testing)
- Very unstable, major bugs expected
- For development team only

**Beta**: `v1.0.0-beta.1` (limited release)
- Feature complete but needs testing
- For beta testers

**Release Candidate**: `v1.0.0-rc.1` (final testing)
- Stable, ready for production
- Final checks before release

**Example Timeline**:
```
v1.0.0-alpha.1  (Week 1)
v1.0.0-alpha.2  (Week 2)
v1.0.0-beta.1   (Week 3)
v1.0.0-beta.2   (Week 4)
v1.0.0-rc.1     (Week 5)
v1.0.0          (Week 6 - PRODUCTION)
```

---

## 4. Complete Git Workflow Examples

### Example 1: Developing a New Feature

```bash
# 1. Ensure develop is up to date
git checkout develop
git pull origin develop

# 2. Create feature branch
git checkout -b feature/church-directory

# 3. Work on feature (multiple commits)
git add app/churches/page.tsx
git commit -m "Add church directory page structure"

git add components/ChurchMap.tsx
git commit -m "Add interactive map with Mapbox"

git add lib/supabase-queries.ts
git commit -m "Add Supabase queries for church data"

# 4. Push feature branch regularly (backup + CI)
git push -u origin feature/church-directory

# 5. Keep feature branch updated with develop
git checkout develop
git pull origin develop
git checkout feature/church-directory
git merge develop  # Or: git rebase develop (cleaner history)

# 6. When feature complete, create Pull Request
gh pr create \
  --base develop \
  --head feature/church-directory \
  --title "Implement church directory with map" \
  --body "Closes #12. Implements user story 1.1 and 1.3"

# 7. After PR approved and merged
git checkout develop
git pull origin develop
git branch -d feature/church-directory  # Delete local branch
```

---

### Example 2: Preparing a Release

```bash
# 1. Ensure develop is ready
git checkout develop
git pull origin develop
# Run full test suite
npm test
npm run build

# 2. Create release branch
git checkout -b release/v1.1.0

# 3. Update version numbers
# Edit package.json: "version": "1.1.0"
# Edit CHANGELOG.md: Add release notes

# 4. Commit version bump
git add package.json CHANGELOG.md
git commit -m "Bump version to 1.1.0"

# 5. Final testing on release branch
npm test
npm run build
# Manual QA testing

# 6. Fix any bugs found
git add .
git commit -m "Fix bug in church search filter"

# 7. Merge to main (production)
git checkout main
git pull origin main
git merge --no-ff release/v1.1.0
git tag -a v1.1.0 -m "Release version 1.1.0: Added transcription and Bible references"
git push origin main --tags

# 8. Merge back to develop (important!)
git checkout develop
git merge --no-ff release/v1.1.0
git push origin develop

# 9. Delete release branch
git branch -d release/v1.1.0

# 10. Deploy to production (automated or manual)
# Vercel/Railway/Fly.io will auto-deploy from main branch

# 11. Create GitHub Release
gh release create v1.1.0 \
  --title "v1.1.0: Transcription & Bible References" \
  --notes-file RELEASE_NOTES.md
```

---

### Example 3: Emergency Hotfix

```bash
# 1. Critical bug found in production!
# Start hotfix from main
git checkout main
git pull origin main
git checkout -b hotfix/v1.0.1-xss-vulnerability

# 2. Fix the bug quickly
git add components/SermonPlayer.tsx
git commit -m "Fix XSS vulnerability in sermon player"

# 3. Test the fix
npm test
npm run build

# 4. Merge to main (production)
git checkout main
git merge --no-ff hotfix/v1.0.1-xss-vulnerability
git tag -a v1.0.1 -m "Hotfix: Security vulnerability in sermon player"
git push origin main --tags

# 5. Merge to develop (so fix is in future releases)
git checkout develop
git merge --no-ff hotfix/v1.0.1-xss-vulnerability
git push origin develop

# 6. Delete hotfix branch
git branch -d hotfix/v1.0.1-xss-vulnerability

# 7. Deploy immediately to production
# Monitor for issues
```

---

## 5. Commit Message Guidelines

### Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types
- **feat**: New feature
- **fix**: Bug fix
- **docs**: Documentation changes
- **style**: Code style changes (formatting, no logic change)
- **refactor**: Code refactoring (no feature/bug change)
- **perf**: Performance improvements
- **test**: Adding or updating tests
- **chore**: Build process, dependencies, tooling
- **ci**: CI/CD changes

### Examples

#### Good Commit Messages ✅

```bash
git commit -m "feat(church-directory): Add interactive map with Mapbox"

git commit -m "fix(sermon-player): Prevent crash when audio URL is null"

git commit -m "docs(architecture): Add n8n workflow diagrams"

git commit -m "refactor(api): Extract Supabase queries into separate module"

git commit -m "perf(search): Optimize full-text search with PostgreSQL indices"

git commit -m "test(bible-extractor): Add unit tests for reference parsing"
```

#### Bad Commit Messages ❌

```bash
git commit -m "fixed stuff"  # ❌ Too vague

git commit -m "WIP"  # ❌ Work in progress (don't commit WIP to develop/main)

git commit -m "asdfasdf"  # ❌ Meaningless

git commit -m "Updated files"  # ❌ Not descriptive

git commit -m "Final version"  # ❌ Confusing (what version?)
```

---

## 6. Pull Request Process

### Creating a Pull Request

```bash
# Using GitHub CLI
gh pr create \
  --base develop \
  --head feature/church-directory \
  --title "Implement church directory with map" \
  --body "$(cat <<EOF
## Description
Implements the church directory page with interactive map showing all churches.

## Related Issues
Closes #12
Related to #15

## Changes
- Added church directory page at /churches
- Integrated Mapbox for interactive map
- Added search and filter functionality
- Created Supabase queries for church data

## Testing
- [x] Manual testing completed
- [x] Mobile responsive
- [x] Accessibility checks
- [x] Cross-browser testing

## Screenshots
[Attach screenshots]
EOF
)"
```

### PR Review Checklist

**Reviewer Checklist**:
- [ ] Code follows project style guidelines
- [ ] No obvious bugs or errors
- [ ] Logic is clear and well-documented
- [ ] Tests added/updated (if applicable)
- [ ] No security vulnerabilities introduced
- [ ] Performance considerations addressed
- [ ] Mobile responsive (if UI changes)
- [ ] Accessibility standards met (WCAG 2.1)
- [ ] No console errors or warnings
- [ ] Database migrations included (if schema changes)

**Author Checklist** (before requesting review):
- [ ] Self-review completed
- [ ] All tests passing locally
- [ ] Branch updated with latest develop
- [ ] No merge conflicts
- [ ] Documentation updated
- [ ] CHANGELOG.md updated (if user-facing)
- [ ] Screenshots added (if UI changes)

---

## 7. Branch Protection Rules

### `main` Branch Protection

**Settings**:
```yaml
Protect matching branches: main

✅ Require pull request reviews before merging
   - Required approving reviews: 1
   - Dismiss stale reviews when new commits are pushed

✅ Require status checks to pass before merging
   - Require branches to be up to date
   - Status checks: CI Tests, Build, Lint

✅ Require conversation resolution before merging

✅ Require linear history (no merge commits from feature branches)

✅ Do not allow bypassing the above settings

✅ Restrict who can push to matching branches
   - Allowed: Maintainers only
```

### `develop` Branch Protection

**Settings**:
```yaml
Protect matching branches: develop

✅ Require pull request reviews before merging
   - Required approving reviews: 1 (can be less strict than main)

✅ Require status checks to pass before merging
   - Status checks: CI Tests, Build

✅ Allow force pushes: No

✅ Allow deletions: No
```

---

## 8. CI/CD Integration

### GitHub Actions Workflow

**File**: `.github/workflows/ci.yml`

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [develop, main]
  pull_request:
    branches: [develop, main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build

  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Staging
        run: |
          # Deploy to staging environment (Vercel preview, etc.)

  deploy-production:
    if: github.ref == 'refs/heads/main'
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to Production
        run: |
          # Deploy to production (Vercel, Railway, Fly.io)
```

---

## 9. Git Aliases (Productivity Boosters)

Add to `~/.gitconfig`:

```ini
[alias]
    # Branch management
    co = checkout
    cob = checkout -b
    br = branch
    brd = branch -d

    # Commit shortcuts
    cm = commit -m
    ca = commit --amend
    cane = commit --amend --no-edit

    # Status and logs
    st = status -sb
    lg = log --oneline --graph --decorate --all
    last = log -1 HEAD

    # Sync with remote
    pom = pull origin main
    pod = pull origin develop
    poh = push origin HEAD
    poh-f = push origin HEAD --force-with-lease

    # Cleanup
    prune-branches = !git branch --merged | grep -v '\\*\\|main\\|develop' | xargs -n 1 git branch -d

    # Rebase shortcuts
    rb = rebase
    rbd = rebase develop
    rbc = rebase --continue
    rba = rebase --abort
```

**Usage**:
```bash
git co develop          # Checkout develop
git cob feature/new     # Create and checkout new branch
git cm "Add feature"    # Commit with message
git lg                  # Pretty log graph
git prune-branches      # Delete merged branches
```

---

## 10. CHANGELOG Format

**File**: `CHANGELOG.md`

```markdown
# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]
### Added
- Feature in development

## [1.1.0] - 2025-01-15
### Added
- Sermon transcription with Whisper
- Bible reference extraction from transcripts
- Search sermons by Bible passage
- Transcript viewer with timestamps

### Changed
- Improved search performance with PostgreSQL indices
- Updated church detail page layout

### Fixed
- Fixed audio player crash when sermon has no audio file
- Fixed map marker clustering on mobile

### Security
- Updated dependencies to patch vulnerabilities

## [1.0.0] - 2024-12-20
### Added
- Church directory with interactive map
- RSS feed ingestion (daily automated)
- Sermon search and browsing
- Audio player
- Admin dashboard

## [0.1.0] - 2024-11-15
### Added
- Initial MVP with basic RSS parsing
- Proof of concept notebook
```

---

## 11. Tag Management

### Creating Tags

```bash
# Lightweight tag (simple reference)
git tag v1.0.0

# Annotated tag (recommended - includes message, author, date)
git tag -a v1.0.0 -m "Release version 1.0.0: MVP launch"

# Tag a specific commit
git tag -a v1.0.0 abc1234 -m "Release version 1.0.0"

# Push tags to remote
git push origin v1.0.0          # Push single tag
git push origin --tags           # Push all tags
```

### Viewing Tags

```bash
# List all tags
git tag

# List tags with pattern
git tag -l "v1.*"

# Show tag details
git show v1.0.0
```

### Deleting Tags

```bash
# Delete local tag
git tag -d v1.0.0

# Delete remote tag
git push origin --delete v1.0.0
```

---

## 12. Common Workflows Summary

### Daily Development Workflow
```bash
# Morning: Start work
git checkout develop
git pull origin develop
git checkout -b feature/new-feature

# During day: Commit frequently
git add .
git commit -m "feat(scope): Description"
git push origin feature/new-feature

# End of day: Push progress
git push origin feature/new-feature

# When complete: Create PR
gh pr create --base develop
```

### Weekly Release Workflow
```bash
# Every Friday: Release to production
git checkout -b release/v1.x.0 develop
# Update versions, changelog
git commit -m "Bump version to 1.x.0"
git checkout main
git merge --no-ff release/v1.x.0
git tag -a v1.x.0 -m "Release notes"
git push origin main --tags
git checkout develop
git merge --no-ff release/v1.x.0
git push origin develop
```

---

## Summary: Quick Reference

| Branch | Purpose | Branch From | Merge To | Lifetime |
|--------|---------|-------------|----------|----------|
| `main` | Production | - | - | Permanent |
| `develop` | Integration | `main` | - | Permanent |
| `feature/*` | New features | `develop` | `develop` | Days-Weeks |
| `bugfix/*` | Bug fixes | `develop` | `develop` | Hours-Days |
| `hotfix/*` | Urgent fixes | `main` | `main` + `develop` | Hours |
| `release/*` | Release prep | `develop` | `main` + `develop` | Days |
| `claude/*` | AI assistant work | current | current | Session |

**Version Format**: `MAJOR.MINOR.PATCH` (e.g., v1.2.3)

**Key Commands**:
```bash
# Create feature
git checkout -b feature/name develop

# Create release
git checkout -b release/v1.0.0 develop

# Create hotfix
git checkout -b hotfix/v1.0.1 main

# Merge with no fast-forward (preserves history)
git merge --no-ff branch-name

# Tag release
git tag -a v1.0.0 -m "Message"
```

This strategy ensures organized development, clear release process, and easy rollback if needed! 🚀
