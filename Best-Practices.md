# Best Practices

Modern best practices for using Jujutsu effectively in 2025.

## General Principles

### 1. Describe Changes Upfront

Instead of committing then describing:

```bash
# Good: Describe what you're going to do
jj new -m "Add user authentication with JWT"
# Now implement it
# Changes automatically go into described commit

# Avoid: Generic or missing descriptions
jj new  # (no description)
# Work...
# Later: "Uh, what was this again?"
```

**Why**: Clear intent before implementation leads to better organized changes.

### 2. Keep Working Copy Clean

Your working copy should always represent your current focus:

```bash
# Good: Create new change when switching focus
jj new -m "Fix bug #123"

# Avoid: Accumulating unrelated changes in working copy
# editing file A for bug fix
# editing file B for feature
# editing file C for refactor
# All mixed together in @
```

### 3. Leverage Automatic Rebasing

Don't fear editing old commits:

```bash
# Good: Fix mistakes anywhere in history
jj edit @---  # Three commits back
# Fix the mistake
# Descendants automatically rebase!

# Avoid: Working around mistakes with fixup commits
jj new -m "Oops fix the thing from 3 commits ago"
# Creates messy history
```

### 4. Use `jj undo` Freely

The operation log is your safety net:

```bash
# Good: Experiment and undo if wrong
jj rebase -s <big-stack> -d main
# Uh oh, that broke things
jj undo
# Try different approach

# Avoid: Fear of trying things
# Hesitating to rebase because it might break
```

### 5. Small, Focused Commits

Each commit should have one logical purpose:

```bash
# Good: Separate commits
jj new -m "Add User model"
jj new -m "Add User controller"
jj new -m "Add User views"

# Avoid: Everything in one commit
jj new -m "Add complete user feature"
# 50 files changed across frontend, backend, tests, docs...
```

## Commit Organization

### Logical Grouping

```bash
# Group related changes together
jj new -m "Database: Add user table schema"
jj new -m "Database: Add user migration"
jj new -m "API: Add user endpoints"
jj new -m "API: Add user validation"
jj new -m "Frontend: Add user registration form"
```

### Test with Each Commit

Each commit should pass tests:

```bash
# After each commit
jj new -m "Add feature X"
# Implement feature
./run-tests.sh  # Should pass!

# If tests fail:
# Fix in the same commit (automatic!)
# Or split into multiple commits
```

### Use `jj split` Generously

When a commit gets too large:

```bash
# Large commit
jj log
# @  Big commit with 20 files

# Split it
jj split
# Separate by:
# - Feature vs tests
# - Different components
# - Setup vs implementation
```

## Branching and Bookmarks

### Bookmarks for Remote Only

Create bookmarks when you need to push:

```bash
# Good: Create bookmark when ready to share
jj new main@origin -m "Feature work"
# Work...
# Ready to push:
jj bookmark create my-feature
jj git push --bookmark my-feature

# Avoid: Creating bookmark first
jj bookmark create my-feature  # Too early!
jj new main@origin -m "Feature work"
```

### Meaningful Bookmark Names

```bash
# Good: Descriptive names
jj bookmark create feature/user-auth
jj bookmark create bugfix/login-timeout
jj bookmark create refactor/cleanup-db-layer

# Avoid: Generic names
jj bookmark create feature  # Which feature?
jj bookmark create wip      # What is it?
jj bookmark create test     # Don't push this!
```

### Clean Up Old Bookmarks

```bash
# After PR merges
jj git fetch
jj bookmark delete my-feature

# Or forget entirely (local + stop tracking)
jj bookmark forget my-feature
```

## Working with Remotes

### Rebase Before Pushing

```bash
# Good workflow:
jj git fetch
jj rebase -d main@origin  # Update to latest
# Resolve conflicts if any
./run-tests.sh            # Ensure tests pass
jj git push --bookmark feature

# Avoid: Pushing outdated work
jj git push --bookmark feature
# Then finding conflicts on remote
```

### Force Push Carefully

```bash
# Good: Force push only your branches
jj git push --bookmark my-feature --force  # OK if it's your branch

# Avoid: Force pushing shared branches
jj git push --bookmark main --force  # ❌ Never!
```

### Communicate Rebases

```bash
# When rebasing shared branches:
# 1. Announce to team
# 2. Ensure others have pushed
# 3. Rebase
# 4. Force push
# 5. Notify team to rebase their work
```

## Conflict Resolution

### Resolve Incrementally

```bash
# Good: Bottom-up resolution
jj log
# @  C (conflict)
# ◉  B (conflict)
# ◉  A (no conflict)

jj edit <B-id>
jj resolve  # Resolve B
jj edit <C-id>
jj resolve  # Resolve C (easier now)

# Avoid: Random order
jj edit <C-id>
jj resolve  # Harder without B resolved
```

### Test After Resolving

```bash
# Good:
jj resolve
# Fix conflicts...
./run-tests.sh  # Verify resolution is correct

# Avoid:
jj resolve
# Fix conflicts...
jj git push  # Hope it works!
```

### Use `jj absorb` for Distributed Fixes

```bash
# Good: Make fixes in working copy, absorb later
# Fix typo in commit A's file
# Fix bug in commit B's file
# Fix formatting in commit C's file
jj absorb  # Automatically puts each fix in right commit

# Avoid: Editing each commit individually
jj edit <A-id>  # Fix A
jj edit <B-id>  # Fix B
jj edit <C-id>  # Fix C
# Tedious!
```

## Team Collaboration

### Clear Commit Messages

```bash
# Good: Descriptive and contextual
jj new -m "Fix login timeout by increasing session TTL to 30 minutes"

# Add more detail in description
jj describe
# Opens editor:
# Fix login timeout by increasing session TTL to 30 minutes
#
# Users were being logged out during long operations.
# Increased session timeout from 15 to 30 minutes.
#
# Fixes: #123

# Avoid:
jj new -m "fix stuff"
jj new -m "wip"
jj new -m "more changes"
```

### Reviewable Commits

Make commits easy to review:

```bash
# Good: Logical progression
1. Add User model
2. Add User validation
3. Add User controller
4. Add User tests
# Reviewer can follow the logic

# Avoid: Scattered changes
1. Half of User model + part of controller
2. Rest of model + some tests
3. Controller + UI + docs all mixed
# Reviewer is confused
```

### Stack PRs for Large Features

```bash
# Break large feature into stacked PRs
jj new main@origin -m "PR 1: User model and database"
jj bookmark create pr-1

jj new -m "PR 2: User API endpoints"
jj bookmark create pr-2

jj new -m "PR 3: User frontend"
jj bookmark create pr-3

# Push all
jj git push --all

# Create PRs:
# PR 1: pr-1 → main
# PR 2: pr-2 → pr-1  (depends on PR 1)
# PR 3: pr-3 → pr-2  (depends on PR 2)

# As each merges, rebase others:
jj git fetch
jj rebase -r pr-2 -d main@origin
jj rebase -r pr-3 -d pr-2
```

## Configuration

### Essential Config

```toml
# ~/.config/jj/config.toml
[user]
name = "Your Name"
email = "your.email@example.com"

[ui]
default-command = "log"
editor = "your-preferred-editor"
diff-editor = ":builtin"

[git]
colocate = true

[revsets]
log = "@ | ancestors(immutable_heads().., 5) | heads(immutable_heads())"
immutable_heads = "main@origin | tags()"
```

### Useful Aliases

```toml
[aliases]
l = ["log"]
s = ["status"]
d = ["diff"]
n = ["new"]
sync = ["git", "fetch", "--all-remotes"]
rb = ["rebase", "-d", "main@origin"]

[revset-aliases]
'mine()' = 'author("your.email@example.com")'
'stack()' = 'ancestors(@) & ~ancestors(main@origin)'
```

## Performance

### Keep History Clean

```bash
# Regularly clean up
jj abandon -r 'empty()'  # Remove empty commits
jj bookmark forget old-feature  # Remove old bookmarks

# Operation log auto-cleans after 90 days
# Configure if needed:
jj config set --user operation.gc-after-days 180
```

### Large Repositories

```bash
# For large repos, consider non-colocated
jj git clone https://large-repo.git  # No --colocate

# Or shallow clone
git clone --depth 1 https://large-repo.git
cd repo
jj git init --colocate
```

## Security

### Sign Commits (If Required)

```toml
# ~/.config/jj/config.toml
[signature]
sign-all = true
backend = "gpg"  # or "ssh"
key = "your-key-id"
```

### Don't Commit Secrets

```bash
# Before committing
jj status  # Check what's being committed

# If you see secrets:
# Remove them from files
# Or use .gitignore

# If already committed:
jj edit <commit-with-secret>
# Remove secret
# Descendants auto-rebase

# For pushed commits:
# Contact security team
# Rotate credentials
```

## Daily Workflow Example

### Morning Routine

```bash
# Update from remote
jj git fetch --all-remotes

# Rebase current work
jj rebase -d main@origin

# Check status
jj log
jj status
```

### During Development

```bash
# Start new feature
jj new main@origin -m "Add feature X"

# Work and test frequently
# Edit files...
./run-tests.sh

# Split if getting large
jj split

# Describe properly
jj describe
```

### Before Pushing

```bash
# Ensure up to date
jj git fetch
jj rebase -d main@origin

# Clean up history
jj squash  # If needed
jj split   # If too large

# Final test
./run-tests.sh

# Create bookmark and push
jj bookmark create feature-x
jj git push --bookmark feature-x
```

### After PR Merged

```bash
# Update
jj git fetch

# Clean up
jj bookmark forget feature-x
jj edit main@origin  # Go back to main
```

## Common Patterns

### Pattern: Feature Flag Development

```bash
# Work behind feature flag
jj new main@origin -m "Add feature X (behind flag)"

# Multiple commits, all behind flag
jj new -m "Part 1"
jj new -m "Part 2"
jj new -m "Part 3"

# Enable flag in final commit
jj new -m "Enable feature X"
```

### Pattern: Experimental Branch

```bash
# Try risky refactoring
jj duplicate @  # Backup current state

# Experiment on current
# Try new approach...

# If it works:
jj abandon <backup-id>

# If it fails:
jj edit <backup-id>
jj abandon <experiment-id>
```

### Pattern: Bug Fix in Old Code

```bash
# Find bug in commit B
jj log
# @  D (latest)
# ◉  C
# ◉  B (has bug)
# ◉  A

# Fix it in place
jj edit <B-id>
# Fix bug
# C and D automatically rebase with fix!

# Or duplicate and cherry-pick
jj duplicate <B-id>
# Fix in duplicate
jj rebase -r <duplicate-id> -d @
```

## Anti-Patterns to Avoid

### Don't: Create Bookmarks Too Early

```bash
# ❌ Bad
jj bookmark create feature
jj new -m "Start feature"

# ✓ Good
jj new -m "Start feature"
# ... work ...
jj bookmark create feature  # Only when pushing
```

### Don't: Accumulate WIP Commits

```bash
# ❌ Bad
jj new -m "wip"
jj new -m "more wip"
jj new -m "fix wip"
jj new -m "actually fix wip"
# Messy history

# ✓ Good
jj new -m "Add feature X"
# Keep working in same commit
# Or use jj squash to clean up
```

### Don't: Ignore Conflicts

```bash
# ❌ Bad
jj rebase -d main
jj log  # Shows (conflict)
jj new -m "Continue anyway"
# Conflict still exists

# ✓ Good
jj rebase -d main
jj log  # Shows (conflict)
jj resolve  # Fix it now
# Clean state
```

### Don't: Skip Testing

```bash
# ❌ Bad
jj new -m "Add feature"
# Implement...
jj git push --bookmark feature
# Hope it works!

# ✓ Good
jj new -m "Add feature"
# Implement...
./run-tests.sh  # Test!
jj git push --bookmark feature
```

## Metrics of Good Practice

You're doing well if:
- ✓ Every commit has a clear, descriptive message
- ✓ Each commit passes tests independently
- ✓ History is linear and logical
- ✓ You use `jj undo` without fear
- ✓ You rebase onto main frequently
- ✓ Commits are small and focused
- ✓ You split large commits
- ✓ You clean up bookmarks after merging

You need improvement if:
- ✗ Commits have no/generic descriptions
- ✗ Tests only pass at the end of stack
- ✗ History is confusing or scattered
- ✗ You're afraid to use jj commands
- ✗ Your branch is weeks behind main
- ✗ Every commit touches 20+ files
- ✗ Never use jj split
- ✗ Dozens of old bookmarks exist

## Next Steps

- [Troubleshooting](Troubleshooting.md) - Solve common issues
- [Resources](Resources.md) - Learn more

---

[← Back to Builtin Editor](Builtin-Editor.md) | [Main](README.md) | [Next: Troubleshooting →](Troubleshooting.md)
