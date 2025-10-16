# Git Interoperability

How to use Jujutsu with Git repositories, colocated repos, and seamless integration.

## Colocated Repositories

The recommended way to use jj with Git is **colocated mode**, where `.jj` and `.git` directories coexist.

### What is Colocated Mode?

```
my-repo/
├── .git/          ← Git repository
├── .jj/           ← Jujutsu repository
├── src/
└── README.md

Both directories track the same working copy
Changes sync automatically between jj and git
```

### Benefits

- Use both `jj` and `git` commands in the same repo
- Seamless integration with Git tools and workflows
- Easy migration path - try jj without commitment
- Full compatibility with existing Git infrastructure

### Setting Up Colocated Repository

#### From Existing Git Repository

```bash
# Navigate to your Git repo
cd /path/to/git/repo

# Initialize jj in colocated mode
jj git init --colocate

# Now you can use both!
git status
jj status
```

#### Cloning a Repository

```bash
# Clone with colocated mode
jj git clone --colocate https://github.com/user/repo.git

# Or with SSH
jj git clone --colocate git@github.com:user/repo.git
```

#### Creating New Repository

```bash
# Create directory
mkdir my-project
cd my-project

# Initialize with colocated mode
jj git init --colocate

# Creates both .git and .jj
ls -la
```

### How Colocated Repos Work

#### Automatic Synchronization

Every jj operation automatically syncs with Git:

```bash
# Create a commit with jj
jj new -m "Add feature"
echo "code" > feature.txt

# Git sees it!
git log  # Shows the commit

# Create a commit with git
git checkout -b temp
echo "more" > file.txt
git add file.txt
git commit -m "Git commit"

# jj sees it!
jj log  # Shows the commit
```

#### Bookmark ↔ Branch Mapping

```bash
# jj bookmarks become Git branches
jj bookmark create feature
# Creates git branch "feature"

# Git branches become jj bookmarks
git branch bugfix
# Creates jj bookmark "bugfix"

# Changes sync automatically
jj bookmark list        # Shows both
git branch --list       # Shows both
```

## Using jj and Git Together

### Recommended Approach

**Primary**: Use `jj` for all local operations
**Secondary**: Use `git` only when necessary (e.g., tools that require it)

```bash
# Prefer jj commands:
jj new -m "Feature"      ✓
jj log                   ✓
jj rebase -d main        ✓

# Avoid git commands for changes:
git add file.txt         ✗ (use jj instead)
git commit               ✗ (use jj instead)
git rebase               ✗ (use jj instead)

# Git is OK for:
git push                 ✓ (or use jj git push)
git pull                 ✓ (or use jj git fetch)
git remote               ✓ (or use jj git remote)
```

### Why Prefer jj Commands?

1. **Consistency** - Avoid confusion between two models
2. **Safety** - jj's operation log tracks everything
3. **Power** - jj's features (like auto-rebase) work best with jj commands
4. **Simplicity** - One mental model is easier than two

### When to Use Git Commands

#### Scenario 1: Legacy Tools

```bash
# Some CI/CD tools expect git
git fetch origin
git merge origin/main

# Or configure jj for the tool
jj git fetch
jj rebase -d main@origin
```

#### Scenario 2: Team Members Don't Use jj

```bash
# They push with git
# You can pull with either:
git pull
# or
jj git fetch && jj rebase -d main@origin
```

#### Scenario 3: Git-Specific Features

```bash
# Tags (not yet fully supported in jj)
git tag v1.0.0
git push --tags

# Git hooks
git commit  # Triggers pre-commit hooks
```

## Synchronization Details

### What Gets Synchronized

| Feature | Syncs? | Notes |
|---------|--------|-------|
| Commits | ✓ | Full sync |
| Branches/Bookmarks | ✓ | Bidirectional |
| Tags | Partial | Read-only in jj |
| Remotes | ✓ | Shared |
| Staging area | ✗ | jj doesn't have one |
| Stashes | ✗ | Use jj changes instead |
| Reflog | ✗ | jj has operation log |

### Synchronization Timing

Sync happens automatically:
- Before every jj operation (import from Git)
- After every jj operation (export to Git)

```bash
# Example flow:
git commit -m "Git commit"    # Creates Git commit
jj log                        # Imports and shows it
jj new -m "jj commit"         # Creates jj commit
git log                       # Exports and shows it
```

## Working with Remotes

### Fetching

```bash
# Using jj (recommended)
jj git fetch
jj git fetch --remote origin
jj git fetch --all-remotes

# Using git (also works)
git fetch
git fetch --all
```

### Pushing

```bash
# Using jj (recommended)
jj bookmark create feature
jj git push --bookmark feature

# Using git (also works)
git push origin feature

# Both create the same remote branch
```

### Pulling Changes

jj doesn't have a `pull` command. Instead:

```bash
# The jj way:
jj git fetch
jj rebase -d main@origin

# Or with git:
git pull --rebase

# Both achieve similar results
```

## Common Workflows

### Workflow 1: Pure jj (Recommended)

```bash
# Setup
jj git clone --colocate https://github.com/user/repo.git
cd repo

# Daily work
jj git fetch
jj new main@origin -m "New feature"
# edit files
jj bookmark create feature
jj git push --bookmark feature

# Update from remote
jj git fetch
jj rebase -d main@origin

# Clean up
jj bookmark delete feature
```

### Workflow 2: Hybrid jj/Git

```bash
# Use jj for local work
jj new -m "Feature"
# edit files

# Use git for remote operations (if team prefers)
jj bookmark create feature
git push -u origin feature

# Someone else pushes
git pull
# jj automatically sees the changes
jj log
```

### Workflow 3: Gradual Migration

```bash
# Keep using git for familiar operations
git checkout -b feature
git add .
git commit -m "Work"

# Try jj for specific tasks
jj log              # Better visualization
jj undo             # Easy undo
jj split            # Split commits

# Gradually shift to jj as comfortable
```

## Handling Conflicts

### Colocated Bookmark/Branch Conflicts

Sometimes bookmarks get out of sync:

```bash
# Check for conflicts
jj bookmark list

# Example output:
# feature: conflict
#   ??? abc123 (from jj)
#   ??? def456 (from git)

# Resolve by choosing one:
jj bookmark set feature -r abc123
# or
jj bookmark set feature -r def456
```

### Divergent Change IDs

Can happen when mixing jj and git commands:

```bash
# Prevention: Prefer jj commands for local work
# If it happens:
jj log  # Look for duplicate commits
jj abandon <duplicate>  # Remove duplicates
```

## Caveats and Limitations

### What Doesn't Sync

#### 1. Git Staging Area

```bash
# Don't do this:
git add file.txt  # Staging not visible to jj
jj status         # Won't show staged files

# Do this instead:
# Just edit files (jj tracks automatically)
```

#### 2. Git Stashes

```bash
# Don't do this:
git stash
jj log  # Doesn't show stashes

# Do this instead:
jj new -m "Temporary work"  # Create a change
jj edit <other-change>       # Switch to other work
```

#### 3. Incomplete Git Operations

```bash
# Don't leave git in the middle of:
git rebase  # Incomplete rebase
git merge   # Incomplete merge
git cherry-pick  # Incomplete cherry-pick

# Complete or abort these before using jj
git rebase --abort
# Then use jj
```

#### 4. Tags

```bash
# Tags are read-only in jj
git tag v1.0.0     # Create with git
jj log             # Can see tags
# But can't create/delete tags with jj
```

### Performance Considerations

#### Large Number of Branches

Many Git branches can slow down colocated repos:

```bash
# Clean up old branches
git branch -d old-feature
git remote prune origin

# Or use --no-colocate for better performance
jj git clone https://github.com/user/repo.git  # Non-colocated
```

#### Repository Size

Very large Git repos may be slower:

```bash
# Consider shallow clone
git clone --depth 1 https://github.com/user/repo.git
cd repo
jj git init --colocate
```

## Non-Colocated Repositories

Alternative: Separate `.jj` and `.git` directories.

### When to Use Non-Colocated

- Very large repositories
- Many Git branches (thousands)
- Want complete separation from Git

### Creating Non-Colocated Repo

```bash
# Clone without colocate flag
jj git clone https://github.com/user/repo.git

# Or init without colocate
jj git init

# Structure:
my-repo/
└── .jj/
    └── repo/
        └── store/
            └── git/  ← Git backend is inside .jj
```

### Differences

```bash
# Colocated: Both commands work
git log
jj log

# Non-colocated: Only jj works
jj log     ✓
git log    ✗ (no .git directory)
```

## Migrating from Git to jj

### Step 1: Try Colocated Mode

```bash
cd your-git-repo
jj git init --colocate

# Try jj commands
jj log
jj status

# Still have git as fallback
git log
git status
```

### Step 2: Use jj for New Features

```bash
# Old way (git):
git checkout -b feature
# work
git add .
git commit

# New way (jj):
jj new -m "Feature"
# work (automatically committed)
jj bookmark create feature
jj git push --bookmark feature
```

### Step 3: Gradually Adopt jj Workflows

```bash
# Start using:
jj log        # Instead of git log
jj show       # Instead of git show
jj diff       # Instead of git diff
jj undo       # Instead of git reset/reflog

# Then:
jj split      # No git equivalent
jj absorb     # Better than git commit --fixup
jj rebase     # Automatic descendants rebase
```

### Step 4: Full Migration (Optional)

```bash
# Once comfortable, go full jj
# Keep using colocated mode for compatibility
# Or switch to non-colocated for performance
```

## Best Practices

### DO

✓ Use `jj git init --colocate` for new repos
✓ Prefer `jj` commands for all local operations
✓ Use `jj undo` frequently
✓ Keep Git branches/bookmarks clean
✓ Use `jj git fetch` instead of `git pull`
✓ Create bookmarks before pushing

### DON'T

✗ Mix `git` and `jj` commands for the same operation
✗ Leave Git in incomplete state (mid-rebase, etc.)
✗ Rely on Git staging in colocated repos
✗ Create too many Git branches (performance)
✗ Use `git stash` (use jj changes instead)

## Troubleshooting

### Problem: Bookmark Conflicts

```bash
# Symptom
jj bookmark list
# feature: conflict

# Solution
jj bookmark set feature -r <correct-revision>
```

### Problem: jj Doesn't See Git Changes

```bash
# Symptom
git commit -m "Work"
jj log  # Doesn't show the commit

# Solution: Trigger import
jj status  # Forces sync
jj log     # Should show now
```

### Problem: Git Doesn't See jj Changes

```bash
# Symptom
jj new -m "Work"
git log  # Doesn't show the commit

# Solution: Ensure bookmark created
jj bookmark create temp
git log  # Should show now
```

### Problem: Slow Performance

```bash
# Cause: Too many Git branches
# Solution 1: Clean up
git branch -d old-feature
git remote prune origin

# Solution 2: Use non-colocated mode
jj git clone https://github.com/user/repo.git  # No --colocate
```

## Configuration

### Configure Default Colocate

```toml
# In ~/.config/jj/config.toml
[git]
colocate = true  # Default to colocated mode
```

### Configure Remote Operations

```toml
[git]
fetch = "all"  # Fetch from all remotes by default
push = "origin"  # Default remote for push
```

## Summary

| Aspect | Colocated | Non-Colocated |
|--------|-----------|---------------|
| **Setup** | `--colocate` flag | Default |
| **Git commands** | Work | Don't work |
| **Compatibility** | High | Medium |
| **Performance** | Can be slower | Faster |
| **Use case** | Most users | Large repos |
| **Migration** | Gradual | Immediate |

## Next Steps

- [Workflows](Workflows.md) - Common jj workflows
- [Configuration](Configuration.md) - Customize your setup
- [Troubleshooting](Troubleshooting.md) - Solve common issues

---

[← Back to Git Comparison](Git-Comparison.md) | [Main](README.md) | [Next: Workflows →](Workflows.md)
