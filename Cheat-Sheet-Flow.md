# Workflow Cheat Sheet: From Multiple Clones to Single Repository

How to replace your multi-clone Git workflow with a streamlined Jujutsu workflow.

## Your Current Git Workflow

### The Problem: Managing 6 Repository Clones

```
~/projects/
├── my-api_01/  ← Working on feature-auth
├── my-api_02/  ← Working on feature-payments
├── my-api_03/  ← Free (looking for this one!)
├── my-api_04/  ← Working on bugfix-login
├── my-api_05/  ← Stale branch, needs cleanup
└── my-api_06/  ← Working on refactor-db
```

### Current Workflow Steps

1. **Find Free Clone**: Search through 6 directories to find one without active work
2. **Update Main**: `cd my-api_03 && git checkout main && git pull --rebase`
3. **Create Branch**: `git checkout -b feature-new-thing`
4. **Start Development**: Work on feature
5. **Interruption!** High priority request comes in → Go back to step 1
6. **Resume Work**: Try to remember which clone has which feature
7. **Repeat**: New requests keep coming, more juggling

### Pain Points

- ❌ Maintaining 6 identical repository clones
- ❌ Disk space wasted (6x the repo size)
- ❌ Searching for a "free" clone
- ❌ Forgetting which clone has which feature
- ❌ Stashing or WIP commits to switch features
- ❌ Pull/update each clone separately
- ❌ IDE/editor state scattered across clones

## The Jujutsu Solution: One Repo, Infinite Features

### How It Works

```
~/projects/
└── my-api/  ← Just one repo!
    ├── .git/
    ├── .jj/
    └── All features managed as parallel changes
```

**Key Insight**: In jj, your working directory is always a commit. You can have as many parallel "working directories" as you want - they're just commits!

### Jujutsu Workflow Steps

1. **Start New Feature**: `jj new main@origin -m "Feature: new thing"`
2. **Work on It**: Edit files (automatically in that commit)
3. **Interruption!** `jj new main@origin -m "Urgent: hotfix"` → Instant switch
4. **Resume Work**: `jj edit <feature-id>` → Back to original feature
5. **See All Work**: `jj log` → Visual view of all features

### Advantages

- ✅ Single repository (1x disk space)
- ✅ No searching for "free" clones
- ✅ Instant context switching (no stashing!)
- ✅ See all features at once with `jj log`
- ✅ One `jj git fetch` updates everything
- ✅ Single IDE/editor state

## Command Comparison

### Starting a New Feature

```bash
# Git (old way)
cd ~/projects/
# Search for free clone...
cd my-api_03
git checkout main
git pull --rebase
git checkout -b feature-new-thing

# Jujutsu (new way)
jj new main@origin -m "Feature: new thing"
# That's it! You're working on it now.
```

### Handling Interruption

```bash
# Git (old way)
git add .
git commit -m "WIP: partial work"  # Or git stash
# Search for another free clone...
cd ~/projects/my-api_05
git checkout main
git pull --rebase
git checkout -b hotfix-urgent

# Jujutsu (new way)
jj new main@origin -m "Hotfix: urgent issue"
# Instantly switched! Previous work is safe.
```

### Returning to Previous Work

```bash
# Git (old way)
# Which clone was that in again?
cd ~/projects/my-api_03
git checkout feature-new-thing
# If you did WIP commit:
git reset HEAD~1  # Undo WIP commit
# Or if stashed:
git stash pop

# Jujutsu (new way)
jj log  # See all your features
jj edit <feature-new-thing-id>
# Instantly back to work, no cleanup needed!
```

### Seeing All Active Work

```bash
# Git (old way)
cd ~/projects/my-api_01 && git branch
cd ~/projects/my-api_02 && git branch
cd ~/projects/my-api_03 && git branch
# ... repeat for all 6 clones

# Jujutsu (new way)
jj log
# Shows all features in one view!
```

## Complete Workflow Example

### Initial Setup

```bash
# One-time setup
cd ~/projects/
jj git clone --colocate https://github.com/yourorg/my-api.git
cd my-api

# Configure your identity
jj config set --user user.name "Your Name"
jj config set --user user.email "you@example.com"
```

### Daily Work: Monday Morning

```bash
# See current state
jj log

# Start feature A
jj new main@origin -m "Feature: Add user authentication"

# Work on it
vim src/auth.js
# Files automatically part of this commit!

# View what you've done
jj status
jj diff
```

### Interruption: Urgent Bug Fix

```bash
# Boss: "Drop everything, fix the login bug!"

# Create urgent fix from main (instantly!)
jj new main@origin -m "Hotfix: Login timeout issue"

# Your auth feature is safe, untouched
# You're now working on the hotfix

# Fix the bug
vim src/login.js

# View your work
jj status  # Shows login.js changes

# Ready to push
jj bookmark create hotfix-login
jj git push --bookmark hotfix-login
```

### Resume Original Work

```bash
# Hotfix is done, back to auth feature

# See all your work
jj log
# @  qpvuntsm Hotfix: Login timeout issue
# │ ◉  rwxyzabc Feature: Add user authentication
# ├─╯
# ◉  main@origin

# Switch back to auth feature
jj edit rwxyzabc  # Use the change ID from log

# Continue working where you left off
vim src/auth.js
# No stash to pop, no WIP commit to undo
# Just keep working!
```

### Another Interruption: Feature B Request

```bash
# Product manager: "We need feature B ASAP!"

# Start feature B (also from main)
jj new main@origin -m "Feature: Payment integration"

# Work on it
vim src/payments.js

# View all your parallel work
jj log
# @  mnoprstu Feature: Payment integration
# │ ◉  qpvuntsm Hotfix: Login timeout issue
# │ ◉  rwxyzabc Feature: Add user authentication
# ├─╯
# ◉  main@origin

# Three features, no problem!
```

### Switching Between Features

```bash
# Quick status check on all work
jj log

# Work on auth for a bit
jj edit rwxyzabc
vim src/auth.js

# Switch to payments
jj edit mnoprstu
vim src/payments.js

# Check on hotfix
jj edit qpvuntsm
jj status  # Already pushed? Clean!

# Back to auth
jj edit rwxyzabc

# Each switch is instant - no stashing!
```

### Finishing and Pushing Features

```bash
# Feature A (auth) is done
jj edit rwxyzabc

# Update from main first
jj git fetch
jj rebase -d main@origin

# Create bookmark and push
jj bookmark create feature-auth
jj git push --bookmark feature-auth

# Create PR on GitHub
gh pr create --title "Add user authentication"

# Feature is pushed, but you can keep working on it locally
# Until PR is merged
```

### Cleanup After Merge

```bash
# PR merged!
jj git fetch

# Clean up
jj bookmark delete feature-auth
jj abandon rwxyzabc  # Or keep for reference

# Continue with other features
jj log  # See remaining work
```

## Advanced Scenarios

### Scenario 1: Building on Previous Feature

```bash
# Feature B depends on Feature A (not yet merged)

# Start from Feature A instead of main
jj new rwxyzabc -m "Feature: Extended auth features"

# Work on it
vim src/auth-advanced.js

# View the stack
jj log
# @  zyxwvuts Feature: Extended auth features
# ◉  rwxyzabc Feature: Add user authentication
# ◉  main@origin

# When A merges, rebase B
jj git fetch
jj rebase -r zyxwvuts -d main@origin
```

### Scenario 2: Emergency Hotfix on Production

```bash
# Production is broken!

# Create hotfix from production tag
jj new production-v1.2.3 -m "Hotfix: Critical security patch"

# Fix it
vim src/security.js

# Test and push immediately
jj bookmark create hotfix-security-v1.2.3
jj git push --bookmark hotfix-security-v1.2.3

# Return to your feature work
jj edit <your-feature-id>
```

### Scenario 3: Experimental Approach

```bash
# Not sure if approach will work, want to try both

# Current feature
jj log
# @  rwxyzabc Feature: Add user authentication

# Duplicate it to try different approach
jj duplicate @
# Creates copy of your current work

# Try approach 1 on original
jj edit rwxyzabc
vim src/auth-approach1.js

# Try approach 2 on duplicate
jj edit <duplicate-id>
vim src/auth-approach2.js

# Compare results
jj diff -r rwxyzabc
jj diff -r <duplicate-id>

# Keep better one
jj abandon <worse-approach-id>
```

### Scenario 4: Code Review Feedback

```bash
# Reviewer: "Change approach in Feature A"

# Switch to that feature
jj edit rwxyzabc

# Make changes
vim src/auth.js
# Changes automatically update the commit!

# Push updated version
jj git push --bookmark feature-auth --force

# Reviewer sees updated PR immediately
```

## Quick Reference Card

### Common Operations

| Task | Git (6 clones) | Jujutsu (1 repo) |
|------|---------------|------------------|
| **Start new feature** | Find free clone → checkout main → pull → create branch | `jj new main@origin -m "Feature"` |
| **Handle interruption** | Commit/stash → find free clone → checkout → pull → branch | `jj new main@origin -m "Urgent"` |
| **Resume work** | Find correct clone → checkout branch → possibly unstash | `jj log` → `jj edit <id>` |
| **See all work** | Check each clone's branches | `jj log` |
| **Update from remote** | Pull in each clone | `jj git fetch` (once!) |
| **Switch features** | `cd` to different clone OR stash → checkout | `jj edit <id>` |

### Essential Commands for This Workflow

```bash
# Start new feature from main
jj new main@origin -m "Description"

# See all your features
jj log

# Switch to feature
jj edit <change-id>

# Update from remote
jj git fetch

# Rebase feature onto latest main
jj rebase -d main@origin

# Push feature
jj bookmark create <name>
jj git push --bookmark <name>

# Clean up after merge
jj bookmark delete <name>
jj abandon <change-id>
```

## Migration Strategy

### Phase 1: Try Alongside (Keep Clones)

```bash
# Keep your 6 clones for now
# Add one jj clone
cd ~/projects/
jj git clone --colocate https://github.com/yourorg/my-api.git my-api-jj

# Use jj for NEW features only
# Keep using Git clones for existing work
```

### Phase 2: Move to jj (Keep One Git Clone)

```bash
# Use jj as primary
cd ~/projects/my-api-jj
# All new work here

# Keep my-api_01 as backup
# Delete others: my-api_02 through _06
```

### Phase 3: Full jj (Single Repo)

```bash
# Delete all old clones
rm -rf ~/projects/my-api_0*

# Rename jj repo
mv ~/projects/my-api-jj ~/projects/my-api

# All work in one place!
```

## Pro Tips

### Tip 1: Use Descriptive Change Messages

```bash
# Good: Easy to find in jj log
jj new main@origin -m "Feature: Payment integration with Stripe"

# Avoid: Hard to distinguish
jj new main@origin -m "new feature"
```

### Tip 2: Clean Up Regularly

```bash
# After PR merges
jj git fetch
jj abandon <merged-change-id>
jj bookmark delete <merged-bookmark>

# Keep log clean
```

### Tip 3: Use Aliases

```bash
# Add to ~/.config/jj/config.toml
[aliases]
new-feature = ["new", "main@origin", "-m"]

# Then:
jj new-feature "Feature: whatever"
```

### Tip 4: Visual Overview Alias

```bash
# Add to config:
[aliases]
active = ["log", "-r", "mine() & ~ancestors(main@origin)"]

# See only your active features:
jj active
```

### Tip 5: Bookmark Only When Pushing

```bash
# Don't create bookmarks immediately
jj new main@origin -m "Feature"

# Only create when ready to push
jj bookmark create feature-name
jj git push --bookmark feature-name
```

## Troubleshooting

### Problem: "Which change was that feature again?"

**Solution**: Use descriptive messages and `jj log`

```bash
jj log
# Easy to scan descriptions
```

### Problem: "I want to see only MY unmerged work"

**Solution**: Use revset

```bash
jj log -r 'mine() & ~ancestors(main@origin)'
```

### Problem: "Made changes in wrong feature"

**Solution**: Use `jj squash` or `jj move`

```bash
# Move changes to correct feature
jj edit <correct-feature-id>
jj squash --from @ --into <correct-feature-id>
```

### Problem: "Lost track of a change"

**Solution**: Operation log saves everything

```bash
jj op log
# Find the change
jj op restore <operation-id>
```

## Comparison: Disk Space Savings

### Git (6 clones)

```
~/projects/
├── my-api_01/  ← 500 MB
├── my-api_02/  ← 500 MB
├── my-api_03/  ← 500 MB
├── my-api_04/  ← 500 MB
├── my-api_05/  ← 500 MB
└── my-api_06/  ← 500 MB
Total: 3 GB
```

### Jujutsu (1 repo)

```
~/projects/
└── my-api/     ← 500 MB
Total: 500 MB
```

**Savings: 2.5 GB** (plus infinite parallel features!)

## Summary

### What You Gain

✅ **One repository** instead of six
✅ **Instant context switching** with `jj edit`
✅ **No stashing** - working copy is always a commit
✅ **See all features** at once with `jj log`
✅ **Automatic rebasing** when editing history
✅ **Undo anything** with `jj undo`
✅ **Simpler mental model** - changes are mutable
✅ **Disk space savings** - 83% less space used

### What Changes

- Think in "changes" not "branches"
- Create bookmarks only for pushing
- Use `jj edit` instead of `git checkout`
- Embrace mutable history (locally)

### Bottom Line

**Before**: Maintain 6 clones, search for free ones, stash/commit to switch
**After**: One repo, infinite parallel features, instant switching

## Next Steps

1. **Read**: [Getting Started](Getting-Started.md)
2. **Learn**: [Core Concepts](Core-Concepts.md)
3. **Reference**: [Git Comparison](Git-Comparison.md)
4. **Practice**: Try jj on a test repo first
5. **Migrate**: Phase 1 → Phase 2 → Phase 3

---

[Main](README.md) | [Getting Started →](Getting-Started.md)
