# Common Workflows

Practical patterns and workflows for using Jujutsu in real-world scenarios.

## Daily Development Workflows

### Workflow 1: Simple Feature Development

The most basic workflow for adding a feature:

```bash
# 1. Start from main
jj git fetch
jj new main@origin -m "Add user authentication"

# 2. Implement the feature
# Edit files...
echo "auth code" > src/auth.rs

# 3. Test
cargo test

# 4. Push
jj bookmark create auth-feature
jj git push --bookmark auth-feature

# 5. Create PR (using GitHub CLI or web)
gh pr create --title "Add user authentication"
```

### Workflow 2: Incremental Development

Building a feature across multiple commits:

```bash
# 1. Create first commit
jj new main@origin -m "Add auth module structure"
# Create file structure
mkdir src/auth
touch src/auth/mod.rs

# 2. Create second commit (child of first)
jj new -m "Implement login function"
# Implement login
echo "fn login() {}" > src/auth/login.rs

# 3. Create third commit
jj new -m "Add password validation"
# Add validation
echo "fn validate() {}" > src/auth/validate.rs

# 4. View your stack
jj log
# @  validate   Add password validation
# ◉  login      Implement login function
# ◉  structure  Add auth module structure
# ◉  main@origin

# 5. Push entire stack
jj bookmark create auth-stack
jj git push --bookmark auth-stack
```

### Workflow 3: Parallel Features

Work on multiple features simultaneously:

```bash
# Start from main
jj git fetch
jj new main@origin -m "Feature A"
# Work on A...

# Start parallel feature (also from main)
jj new main@origin -m "Feature B"
# Work on B...

# View both features
jj log
# @  Feature B
# │ ◉  Feature A
# ├─╯
# ◉  main@origin

# Switch between them
jj edit <feature-a-id>  # Work on A
jj edit <feature-b-id>  # Work on B

# Push separately
jj bookmark create feature-a -r <feature-a-id>
jj bookmark create feature-b -r <feature-b-id>
jj git push --all
```

## Code Review Workflows

### Workflow 4: Addressing Review Feedback

Incorporating feedback from code review:

```bash
# You have a PR with commits A → B → C
jj log
# @  C  Add tests
# ◉  B  Implement feature
# ◉  A  Add structure

# Reviewer says: "Fix B"
# Edit that specific commit
jj edit <B-id>
# Make changes
echo "improved code" > file.rs
# Changes are automatically in B!

# C is automatically rebased!
jj log
# @  C'  Add tests (rebased)
# ◉  B'  Implement feature (updated)
# ◉  A   Add structure

# Push updated commits
jj git push --bookmark feature --force
```

### Workflow 5: Interactive Squashing

Cleaning up commits before pushing:

```bash
# You have messy WIP commits
jj log
# @  WIP: fix tests
# ◉  WIP: more work
# ◉  WIP: start feature
# ◉  main@origin

# Squash them all together
jj squash -r @-     # Squash middle commit
jj squash -r @-     # Squash next one
# Now you have one clean commit

# Or use interactive mode
jj squash -i        # Choose what to squash

# Describe properly
jj describe -m "Add complete feature with tests"

# Push clean history
jj bookmark create feature
jj git push --bookmark feature
```

### Workflow 6: Split Large Commit

Breaking down a large commit:

```bash
# You have one big commit
jj log
# @  Big commit with many changes

# Split it
jj split
# Interactive editor opens
# Select files for first commit
# Remaining files go to second commit

# Result
jj log
# @  Part 2 of changes
# ◉  Part 1 of changes

# Describe each part
jj describe -m "Add core functionality"
jj edit @-
jj describe -m "Add tests and docs"
```

## Updating from Main

### Workflow 7: Rebasing Feature Branch

Keep your feature up to date with main:

```bash
# You're working on a feature
jj log
# @  My feature
# ◉  Old main
# ◉  ...

# Main has advanced
jj git fetch
jj log -r 'main@origin'
# Shows new commits

# Rebase onto latest main
jj rebase -d main@origin

# Result
jj log
# @  My feature (rebased)
# ◉  New main commits
# ◉  ...

# If conflicts, they're marked but don't block
jj log
# @  My feature (conflict)
# Resolve when ready
jj resolve
```

### Workflow 8: Updating Feature Stack

Rebase an entire stack of commits:

```bash
# You have a stack
jj log
# @  Feature C
# ◉  Feature B
# ◉  Feature A
# ◉  Old main

# Rebase entire stack
jj rebase -s <feature-a-id> -d main@origin

# All three commits rebased
jj log
# @  Feature C (rebased)
# ◉  Feature B (rebased)
# ◉  Feature A (rebased)
# ◉  New main
```

## Fixing Mistakes

### Workflow 9: Fixing Old Commit

Fix a bug in an earlier commit:

```bash
# Bug is in commit B
jj log
# @  D  Latest work
# ◉  C  More work
# ◉  B  Has a bug
# ◉  A  Initial commit

# Edit that commit
jj edit <B-id>
# Fix the bug
echo "fixed code" > buggy-file.rs

# Return to tip
jj edit @  # Or use the change ID of D

# C and D are automatically rebased with fix!
jj log
# @  D'  Latest work
# ◉  C'  More work
# ◉  B'  Has a bug (fixed)
# ◉  A   Initial commit
```

### Workflow 10: Absorb Small Fixes

Automatically distribute fixes to the right commits:

```bash
# You have a stack
jj log
# @  Working copy (empty)
# ◉  C  Add feature
# ◉  B  Add tests
# ◉  A  Add structure

# Make small fixes across files
echo "fix in A's file" > file-a.rs
echo "fix in B's file" > file-b.rs
echo "fix in C's file" > file-c.rs

# Absorb automatically distributes them!
jj absorb

# Result: Each fix goes to the commit that last modified that line
jj log
# @  Working copy (empty)
# ◉  C'  Add feature (with fix)
# ◉  B'  Add tests (with fix)
# ◉  A'  Add structure (with fix)
```

### Workflow 11: Undo Mistake

Undo a bad operation:

```bash
# You did something wrong
jj squash  # Oops! Didn't mean to squash that

# Undo it
jj undo

# Back to previous state!

# Made multiple mistakes?
jj undo
jj undo
jj undo

# Or restore to specific operation
jj op log  # Find the good state
jj op restore <operation-id>
```

## Collaboration Workflows

### Workflow 12: Continuing Someone Else's Work

Take over a branch from a colleague:

```bash
# Fetch their work
jj git fetch
jj bookmark track feature@origin

# Create your own commits on top
jj new feature@origin -m "Continue work"
# Make changes

# Push your additions
jj git push --bookmark feature
```

### Workflow 13: Cherry-Pick Changes

Copy commits from another branch:

```bash
# You want commit X from another branch
jj log -r 'all()'  # Find the commit

# Duplicate it
jj duplicate <commit-x-id>

# Rebase the duplicate onto your branch
jj rebase -r <new-id> -d @

# Result: Commit X is now in your branch
```

### Workflow 14: Sharing Work in Progress

Share unfinished work with others:

```bash
# Create a WIP bookmark
jj bookmark create wip/my-feature

# Push it
jj git push --bookmark wip/my-feature

# Continue working
jj new -m "More WIP"
# edit...

# Update bookmark
jj bookmark set wip/my-feature
jj git push --bookmark wip/my-feature --force

# Others can track it
# (On their machine)
jj git fetch
jj bookmark track wip/my-feature@origin
jj new wip/my-feature@origin -m "My additions"
```

## Advanced Workflows

### Workflow 15: Stacked Pull Requests

Create multiple dependent PRs:

```bash
# Create stack of changes
jj new main@origin -m "PR 1: Foundation"
# work...
jj bookmark create pr-1

jj new -m "PR 2: Build on foundation"
# work...
jj bookmark create pr-2

jj new -m "PR 3: Final feature"
# work...
jj bookmark create pr-3

# Push all
jj git push --all

# Create PRs:
# PR 1: pr-1 → main
# PR 2: pr-2 → pr-1
# PR 3: pr-3 → pr-2

# When PR 1 merges, rebase others
jj git fetch
jj rebase -r pr-2 -d main@origin
jj rebase -r pr-3 -d pr-2
jj git push --bookmark pr-2 --force
jj git push --bookmark pr-3 --force
```

### Workflow 16: Experimental Changes

Try something risky without losing work:

```bash
# Duplicate your current state
jj duplicate @
# Creates a backup

# Experiment on current change
# Try risky refactoring...

# Doesn't work? Switch to backup
jj edit <backup-id>

# Or if it works:
jj abandon <backup-id>
```

### Workflow 17: Move Changes Between Branches

Reorganize commits:

```bash
# You have:
jj log
# @  Change C (should be on feature-2)
# ◉  Change B (should be on feature-2)
# ◉  Change A (on feature-1) ✓
# ◉  main

# Rebase B and C to main (parallel to A)
jj rebase -s <B-id> -d main

# Result:
# @  Change C
# ◉  Change B
# │ ◉  Change A
# ├─╯
# ◉  main

# Create separate bookmarks
jj bookmark create feature-2 -r <C-id>
jj bookmark create feature-1 -r <A-id>
```

## Hotfix Workflows

### Workflow 18: Emergency Hotfix

Quickly fix production issue:

```bash
# You're in the middle of feature work
jj log
# @  Feature work (incomplete)
# ◉  More feature work
# ◉  main@origin

# Hotfix from main
jj new main@origin -m "Hotfix: Critical bug"
# Fix the bug
echo "fix" > critical-file.rs

# Test
./run-tests.sh

# Push immediately
jj bookmark create hotfix-123
jj git push --bookmark hotfix-123

# Return to feature work
jj edit <feature-work-id>

# After hotfix merges, rebase feature
jj git fetch
jj rebase -d main@origin
```

### Workflow 19: Backport Fix

Apply fix to multiple branches:

```bash
# You fixed something on main
jj log
# @  Fix bug
# ◉  main@origin

# Backport to release branch
jj duplicate @
jj rebase -r <duplicate-id> -d release-1.0@origin

# Push
jj bookmark create backport-fix -r <duplicate-id>
jj git push --bookmark backport-fix

# Repeat for other release branches
jj duplicate <original-fix-id>
jj rebase -r <new-duplicate-id> -d release-2.0@origin
```

## Conflict Resolution Workflows

### Workflow 20: Resolve Conflicts Incrementally

Handle conflicts in a stack:

```bash
# Rebase creates conflicts
jj rebase -d main@origin

jj log
# @  C (conflict)
# ◉  B (conflict)
# ◉  A (no conflict)
# ◉  main@origin

# Resolve bottom-up
jj edit <A-id>  # No conflict, skip

jj edit <B-id>
jj resolve       # Resolve B's conflicts

jj edit <C-id>
jj resolve       # Resolve C's conflicts

# All resolved!
```

### Workflow 21: Use Absorb After Merge

Fix conflicts with absorb:

```bash
# After merge with conflicts
jj log
# @  Merge (conflict)

# Resolve conflicts manually
# Edit files to fix...

# Create new change with fixes
jj new -m "Conflict resolutions"
# Move changes to appropriate commits

# Absorb distributes fixes
jj absorb

# Squash the now-empty resolution commit
jj abandon @
```

## Testing Workflows

### Workflow 22: Test Each Commit

Ensure each commit in stack works:

```bash
# You have a stack
jj log
# @  C
# ◉  B
# ◉  A
# ◉  main

# Test A
jj edit <A-id>
./run-tests.sh
# Passes ✓

# Test B
jj edit <B-id>
./run-tests.sh
# Fails ✗ - fix it
echo "fix" > file.rs
# Automatically updates B

# Test C
jj edit <C-id>
./run-tests.sh
# Passes ✓

# Return to tip
jj edit @
```

### Workflow 23: Bisect to Find Bug

Find which commit introduced a bug:

```bash
# Binary search through history
jj new <old-good-commit>
./run-tests.sh  # Works

jj new <middle-commit>
./run-tests.sh  # Fails - bug is between old and middle

jj new <quarter-commit>
./run-tests.sh  # Works - bug is between quarter and middle

# Continue until you find the exact commit
# Fix it
jj edit <buggy-commit-id>
echo "fix" > file.rs
# All descendants automatically rebase
```

## Git Integration Workflows

### Workflow 24: Sync with Git Team

Work with team members using Git:

```bash
# They push with git
# You fetch and see their changes
jj git fetch
jj log -r 'remote_bookmarks()'

# Create your changes
jj new main@origin -m "My work"
# edit...

# They push more changes
jj git fetch

# Rebase your work
jj rebase -d main@origin

# Push (they see it as normal git commit)
jj bookmark create my-work
jj git push --bookmark my-work
```

### Workflow 25: Migrate Project to jj

Gradually introduce jj to team:

```bash
# Phase 1: You use jj, push via Git
jj git clone --colocate <repo>
jj new main@origin -m "Work"
# edit...
jj bookmark create feature
jj git push --bookmark feature  # Team sees normal Git branch

# Phase 2: Document jj usage
# Add .jj/config.toml to repo
# Add docs/jj-guide.md

# Phase 3: Others try jj
# They can use colocated mode
# Gradually adopt jj commands

# Phase 4: Full jj workflow
# Everyone uses jj
# Git is just the backend
```

## Productivity Workflows

### Workflow 26: Quick Context Switching

Rapidly switch between tasks:

```bash
# Working on feature A
jj log
# @  Feature A work

# Boss: "Drop everything, fix bug!"
jj new main@origin -m "Urgent: Fix bug"
# fix bug
jj bookmark create hotfix
jj git push --bookmark hotfix

# Back to feature A
jj edit <feature-a-id>
# Continue where you left off

# No stashing needed!
```

### Workflow 27: Experiment Safely

Try different approaches:

```bash
# Current state
jj log
# @  Working solution

# Try alternative approach
jj duplicate @
jj describe -m "Alternative approach"
# Try different implementation

# Compare both
jj diff -r <solution-1-id>
jj diff -r <solution-2-id>

# Keep better one
jj abandon <worse-id>
```

## Best Practices Summary

### DO

✓ Create descriptive commit messages upfront with `jj new -m`
✓ Use `jj log` frequently to stay oriented
✓ Leverage `jj undo` fearlessly
✓ Split large commits with `jj split`
✓ Use `jj absorb` for automatic fix distribution
✓ Edit history freely before pushing
✓ Test each commit in a stack

### DON'T

✗ Create bookmarks before you need them
✗ Fear editing old commits
✗ Mix `jj` and `git` commands unnecessarily
✗ Leave commits without descriptions
✗ Push before cleaning up history
✗ Ignore conflicts (resolve incrementally)

## Next Steps

- [Advanced Features](Advanced-Features.md) - Power user features
- [Configuration](Configuration.md) - Optimize your workflow
- [Best Practices](Best-Practices.md) - Modern patterns

---

[← Back to Git Interop](Git-Interop.md) | [Main](README.md) | [Next: Conflict Resolution →](Conflict-Resolution.md)
