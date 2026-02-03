# Git Advanced Guide - Complete Deep Dive
## Branching, Merging, Rebasing, Workflows, Troubleshooting

---

# 1. GIT INTERNALS

## How Git Stores Data

```
GIT OBJECT MODEL:
┌─────────────────────────────────────────────────────────────────┐
│ BLOB:     File contents (no filename)                          │
│ TREE:     Directory structure (points to blobs/trees)          │
│ COMMIT:   Snapshot (points to tree, parent commits, metadata)  │
│ TAG:      Named pointer to commit                              │
└─────────────────────────────────────────────────────────────────┘

COMMIT STRUCTURE:
┌─────────────────────────────────────────────────────────────────┐
│ commit abc123                                                   │
│ ├── tree: def456       (root directory snapshot)               │
│ ├── parent: 789xyz     (previous commit)                       │
│ ├── author: John <j@e.com> 1234567890 +0000                    │
│ ├── committer: John <j@e.com> 1234567890 +0000                 │
│ └── message: "Add feature X"                                   │
└─────────────────────────────────────────────────────────────────┘

REFERENCES:
┌─────────────────────────────────────────────────────────────────┐
│ HEAD:        Current commit/branch                             │
│ main:        Branch (pointer to commit)                        │
│ origin/main: Remote tracking branch                            │
│ tags/v1.0:   Tag (permanent pointer)                           │
└─────────────────────────────────────────────────────────────────┘
```

## The Three Trees

```
GIT'S THREE TREES:

┌─────────────────┐    git add     ┌─────────────────┐    git commit   ┌─────────────────┐
│  Working Dir    │ ─────────────► │  Staging Area   │ ─────────────► │    Repository   │
│  (Filesystem)   │                │    (Index)      │                │     (HEAD)      │
└─────────────────┘                └─────────────────┘                └─────────────────┘
        ▲                                                                      │
        └──────────────────────────────────────────────────────────────────────┘
                                    git checkout / restore
```

---

# 2. BRANCHING STRATEGIES

## Git Flow

```
GIT FLOW BRANCHES:
┌─────────────────────────────────────────────────────────────────┐
│ main/master:     Production code                               │
│ develop:         Integration branch                            │
│ feature/*:       New features                                  │
│ release/*:       Release preparation                           │
│ hotfix/*:        Production fixes                              │
└─────────────────────────────────────────────────────────────────┘

WORKFLOW:
         main    develop   feature/x   release/1.0   hotfix/fix
           │        │          │            │            │
           │        ├──────────┤            │            │
           │        │   work   │            │            │
           │        │◄─────────┤ (merge)    │            │
           │        ├──────────────────────►│            │
           │        │                       │ (prep)     │
           │◄───────┼───────────────────────┤ (release)  │
           │        │◄──────────────────────┤            │
           ├────────┼───────────────────────┼───────────►│
           │        │                       │            │ (fix)
           │◄───────┼───────────────────────┼────────────┤
           │        │◄──────────────────────┼────────────┤

PROS: Clear structure, parallel development
CONS: Complex, many branches
BEST FOR: Large teams, scheduled releases
```

## Trunk-Based Development

```
TRUNK-BASED DEVELOPMENT:
┌─────────────────────────────────────────────────────────────────┐
│ main:             Single long-lived branch                     │
│ feature/*:        Short-lived (max 2 days)                     │
│ release/*:        Optional, for stabilization                  │
└─────────────────────────────────────────────────────────────────┘

WORKFLOW:
         main          feature/x
           │               │
           ├───────────────┤ (short-lived)
           │     work      │
           │◄──────────────┤ (merge quickly)
           │               
           ├───────────────┤ (another feature)
           │◄──────────────┤

PROS: Simple, continuous integration, fewer conflicts
CONS: Requires feature flags, good CI/CD
BEST FOR: Experienced teams, continuous deployment
```

## GitHub Flow

```
GITHUB FLOW (Simplified Git Flow):
┌─────────────────────────────────────────────────────────────────┐
│ main:        Always deployable                                 │
│ feature/*:   All work happens here                            │
└─────────────────────────────────────────────────────────────────┘

WORKFLOW:
1. Create branch from main
2. Make commits
3. Open Pull Request
4. Review and discuss
5. Deploy to test environment
6. Merge to main
7. Deploy to production

PROS: Simple, clear
BEST FOR: Web apps, continuous deployment
```

---

# 3. MERGING

## Merge Types

```
MERGE STRATEGIES:

Fast-Forward Merge:
┌─────────────────────────────────────────────────────────────────┐
│ Before:                                                         │
│   main ────●────●                                               │
│                  \                                              │
│                   ●────●────● feature                           │
│                                                                 │
│ After (fast-forward):                                           │
│   main ────●────●────●────●────● (no merge commit)              │
└─────────────────────────────────────────────────────────────────┘

Three-Way Merge:
┌─────────────────────────────────────────────────────────────────┐
│ Before:                                                         │
│   main ────●────●────●────●                                     │
│                  \                                              │
│                   ●────●────● feature                           │
│                                                                 │
│ After (merge commit):                                           │
│   main ────●────●────●────●────● (merge commit)                 │
│                  \            /                                 │
│                   ●────●────●                                   │
└─────────────────────────────────────────────────────────────────┘

Squash Merge:
┌─────────────────────────────────────────────────────────────────┐
│ Before:                                                         │
│   main ────●────●                                               │
│                  \                                              │
│                   ●────●────● feature (3 commits)               │
│                                                                 │
│ After (squash):                                                 │
│   main ────●────●────● (1 commit with all changes)              │
└─────────────────────────────────────────────────────────────────┘
```

## Merge Commands

```bash
# Basic merge
git checkout main
git merge feature-branch

# No fast-forward (always create merge commit)
git merge --no-ff feature-branch

# Squash merge
git merge --squash feature-branch
git commit -m "Feature X complete"

# Abort merge (if conflicts)
git merge --abort
```

---

# 4. REBASING

## What Is Rebase?

```
REBASE = Move commits to new base

Before:
  main ────●────●────●────● (new commits)
               \
                ●────●────● feature (your work)

After rebase:
  main ────●────●────●────●
                          \
                           ●────●────● feature (replayed)

ADVANTAGE: Clean, linear history
DANGER: Rewrites history (never rebase shared branches!)
```

## Rebase Commands

```bash
# Basic rebase
git checkout feature
git rebase main

# Interactive rebase (edit commits)
git rebase -i HEAD~5  # Last 5 commits

# Interactive rebase options:
# pick   = keep commit as is
# reword = change commit message
# edit   = stop to amend commit
# squash = combine with previous commit
# fixup  = like squash but discard message
# drop   = remove commit

# Abort rebase
git rebase --abort

# Continue after resolving conflicts
git rebase --continue

# Skip current commit
git rebase --skip
```

## Rebase vs Merge

```
WHEN TO USE MERGE:
- Preserving complete history
- Shared/public branches
- Feature branches merged to main

WHEN TO USE REBASE:
- Cleaning up local commits before PR
- Updating feature branch with latest main
- Private/local branches only

GOLDEN RULE: Never rebase commits that exist outside your repository
```

---

# 5. CONFLICT RESOLUTION

## Understanding Conflicts

```
CONFLICT MARKERS:
<<<<<<< HEAD
Content from current branch
=======
Content from merging branch
>>>>>>> feature-branch

EXAMPLE:
<<<<<<< HEAD
function greet(name) {
    return "Hello, " + name;
}
=======
function greet(name) {
    return `Hello, ${name}!`;
}
>>>>>>> feature-branch
```

## Resolution Steps

```bash
# 1. Identify conflicted files
git status

# 2. Open file and resolve manually
# Remove conflict markers, keep desired code

# 3. Mark as resolved
git add <file>

# 4. Complete merge
git commit  # or git rebase --continue

# Tools for resolution
git mergetool  # Opens configured merge tool

# Configure merge tool
git config --global merge.tool vscode
git config --global mergetool.vscode.cmd 'code --wait $MERGED'
```

---

# 6. ADVANCED COMMANDS

## Cherry-Pick

```bash
# Apply specific commit to current branch
git cherry-pick abc123

# Cherry-pick range
git cherry-pick abc123..def456

# Cherry-pick without committing
git cherry-pick -n abc123

# Cherry-pick with custom message
git cherry-pick -e abc123
```

## Stash

```bash
# Save work in progress
git stash
git stash push -m "Work in progress on feature X"

# List stashes
git stash list

# Apply stash
git stash pop           # Apply and remove
git stash apply         # Apply and keep
git stash apply stash@{2}  # Apply specific stash

# View stash contents
git stash show -p stash@{0}

# Create branch from stash
git stash branch new-branch stash@{0}

# Drop stash
git stash drop stash@{0}
git stash clear  # Remove all stashes
```

## Bisect

```bash
# Find commit that introduced bug
git bisect start
git bisect bad                    # Current commit is bad
git bisect good abc123            # This commit was good

# Git checks out middle commit, you test:
git bisect good  # or
git bisect bad

# Repeat until found

# End bisect
git bisect reset

# Automated bisect
git bisect run ./test-script.sh
```

## Reflog

```bash
# View reference log (safety net)
git reflog

# Recover lost commits
git checkout HEAD@{5}
git cherry-pick HEAD@{5}
git reset --hard HEAD@{5}

# Reflog is local only, expires after 90 days
```

## Reset vs Revert

```bash
# RESET: Move branch pointer (rewrites history)
git reset --soft HEAD~1   # Undo commit, keep changes staged
git reset --mixed HEAD~1  # Undo commit, keep changes unstaged (default)
git reset --hard HEAD~1   # Undo commit, discard changes

# REVERT: Create new commit that undoes changes (safe for shared branches)
git revert abc123
git revert abc123..def456  # Revert range
git revert -n abc123       # Revert without committing
```

---

# 7. WORKING WITH REMOTES

## Remote Commands

```bash
# List remotes
git remote -v

# Add remote
git remote add origin https://github.com/user/repo.git
git remote add upstream https://github.com/original/repo.git

# Remove remote
git remote remove origin

# Rename remote
git remote rename origin upstream

# Update remote URL
git remote set-url origin git@github.com:user/repo.git

# Fetch from remote
git fetch origin
git fetch --all

# Pull (fetch + merge)
git pull origin main
git pull --rebase origin main  # Fetch + rebase instead

# Push
git push origin main
git push -u origin feature   # Set upstream
git push --force-with-lease  # Safe force push
git push --tags              # Push tags
```

## Upstream Tracking

```bash
# Set upstream for current branch
git branch -u origin/main
git push -u origin feature

# View tracking
git branch -vv

# Unset upstream
git branch --unset-upstream
```

---

# 8. GIT HOOKS

## Available Hooks

```
CLIENT-SIDE HOOKS:
┌─────────────────────────────────────────────────────────────────┐
│ pre-commit:       Before commit (lint, test)                   │
│ prepare-commit-msg: After default message, before editor       │
│ commit-msg:       Validate commit message                      │
│ post-commit:      After commit (notification)                  │
│ pre-push:         Before push (test, lint)                     │
│ pre-rebase:       Before rebase                                │
└─────────────────────────────────────────────────────────────────┘

SERVER-SIDE HOOKS:
┌─────────────────────────────────────────────────────────────────┐
│ pre-receive:      Before accepting push                        │
│ update:           Once per branch being pushed                 │
│ post-receive:     After push (CI/CD trigger)                   │
└─────────────────────────────────────────────────────────────────┘
```

## Hook Examples

```bash
# .git/hooks/pre-commit
#!/bin/bash
npm run lint
if [ $? -ne 0 ]; then
  echo "Linting failed. Commit aborted."
  exit 1
fi

# .git/hooks/commit-msg
#!/bin/bash
if ! grep -qE "^(feat|fix|docs|style|refactor|test|chore):" "$1"; then
  echo "Commit message must follow conventional commits format"
  exit 1
fi
```

## Husky (Node.js Projects)

```bash
# Install husky
npm install husky --save-dev
npx husky init

# Add pre-commit hook
echo "npm test" > .husky/pre-commit

# Add commit-msg hook
echo 'npx commitlint --edit "$1"' > .husky/commit-msg
```

---

# 9. TROUBLESHOOTING

## Common Scenarios

### Undo Last Commit
```bash
# Keep changes
git reset --soft HEAD~1

# Discard changes
git reset --hard HEAD~1

# Already pushed (creates revert commit)
git revert HEAD
```

### Fix Last Commit Message
```bash
# Not pushed yet
git commit --amend -m "New message"

# Already pushed (careful - rewrites history)
git commit --amend -m "New message"
git push --force-with-lease
```

### Add Forgotten File to Last Commit
```bash
git add forgotten-file.txt
git commit --amend --no-edit
```

### Recover Deleted Branch
```bash
# Find the commit
git reflog

# Recreate branch
git checkout -b recovered-branch HEAD@{5}
```

### Remove File from Git (Keep Local)
```bash
git rm --cached filename
echo "filename" >> .gitignore
git commit -m "Remove filename from tracking"
```

### Remove Sensitive Data from History
```bash
# Using git filter-repo (recommended)
pip install git-filter-repo
git filter-repo --path secrets.txt --invert-paths

# Force push to all branches
git push --force --all
```

### Fix Detached HEAD
```bash
# Create branch at current position
git checkout -b new-branch

# Or return to branch
git checkout main
```

---

# 10. BEST PRACTICES

```
COMMIT MESSAGES:
┌─────────────────────────────────────────────────────────────────┐
│ Conventional Commits:                                          │
│ feat: Add user authentication                                  │
│ fix: Resolve login timeout issue                               │
│ docs: Update API documentation                                 │
│ refactor: Extract validation logic                             │
│ test: Add unit tests for auth module                          │
│ chore: Update dependencies                                     │
└─────────────────────────────────────────────────────────────────┘

BRANCHING:
┌─────────────────────────────────────────────────────────────────┐
│ ✓ Use descriptive names: feature/user-auth, fix/login-bug     │
│ ✓ Keep branches short-lived                                   │
│ ✓ Delete merged branches                                       │
│ ✓ Pull latest before starting work                            │
└─────────────────────────────────────────────────────────────────┘

WORKFLOW:
┌─────────────────────────────────────────────────────────────────┐
│ ✓ Commit early, commit often                                  │
│ ✓ Write meaningful commit messages                            │
│ ✓ Review your diff before committing                          │
│ ✓ Never commit directly to main                               │
│ ✓ Use pull requests for code review                           │
│ ✓ Keep commits focused (one logical change)                   │
│ ✓ Don't commit generated files                                │
└─────────────────────────────────────────────────────────────────┘
```
