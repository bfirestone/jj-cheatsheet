# Core Concepts

Understanding Jujutsu's mental model - the key to mastering the tool.

## The Big Ideas

### 1. Working Copy as a Commit

**The most important concept**: In jj, your working directory is always a commit.

```
Git:                  Jujutsu:
───────────           ───────────
Commit A              Commit A
Commit B              Commit B
[Staging Area]        Working Copy Commit (@)
[Working Dir]         ↑ Always a commit!
```

**What this means:**
- No staging area (index)
- Changes are immediately part of a commit
- Every edit automatically updates the current commit
- You're never in a "dirty" state

**Example:**
```bash
jj new -m "Add feature"  # Create new commit
echo "code" > file.txt   # Edit files
# ← Changes are already in "Add feature" commit!
```

### 2. Changes vs Commits

Jujutsu uses two identifiers for each commit:

#### Change ID (Persistent)
- Stays the same even when commit content changes
- Survives rebases, amendments, etc.
- Example: `qpvuntsm`
- Used for referring to "the same logical change"

#### Commit ID (Traditional Hash)
- Changes whenever commit content changes
- Like Git's commit hash
- Example: `4e411c1a`
- Used for exact content identification

**Example:**
```bash
$ jj log
@  qpvuntsm ben 2025-10-15 4e411c1a
│  Add feature
   └─ Change ID (persistent)
      └─ Commit ID (changes with content)

# Edit the commit
echo "more code" >> file.txt

$ jj log
@  qpvuntsm ben 2025-10-15 8f522e2b  ← Commit ID changed
│  Add feature                       ← Change ID stayed same!
```

### 3. Automatic Rebasing

When you modify a commit, descendant commits are automatically rebased.

```bash
# Create a stack
jj new -m "A"
echo "a" > a.txt

jj new -m "B"
echo "b" > b.txt

jj new -m "C"
echo "c" > c.txt

# Go back and edit A
jj edit @--  # Two commits back (to A)
echo "modified a" > a.txt

# B and C are automatically rebased!
# No manual rebasing needed
```

**Visualization:**
```
Before:          Edit A:           After:
  C                C'                 C''
  │                │                  │
  B         →      B          →      B'
  │                │                  │
  A                A'                A'
```

### 4. First-Class Conflicts

Conflicts don't block operations - they're part of the history.

```bash
# In Git:
git rebase main
# CONFLICT! Must resolve before continuing
# Working directory is in "rebase in progress" state

# In Jujutsu:
jj rebase -d main
# Rebase completes! Conflicts are marked but don't stop you
# Can continue working, resolve later
```

**Conflict markers in log:**
```bash
$ jj log
@  qpvuntsm (conflict) Add feature
│  ↑ Conflict is visible but doesn't block operations
```

### 5. Immutable History vs Mutable Changes

```
Local (mutable):        Remote (immutable):
─────────────          ──────────────
Can freely edit,       Once pushed,
rebase, squash, etc.   treated as immutable

jj rebase -d ...      jj git push
jj squash                  │
jj edit               ────────────
                      Becomes part
                      of shared history
```

**Configuration to enforce:**
```toml
# In ~/.config/jj/config.toml
revsets.immutable_heads = "main@origin | tags()"
```

## Deep Dive: Working Copy

### Anatomy of the Working Copy

```bash
$ jj status
Working copy : qpvuntsm 4e411c1a (empty) (no description set)
Parent commit: zzzzzzzz 00000000 (empty) (no description set)

┌─────────────────────────────────┐
│ Working Copy Commit             │
│ Change ID: qpvuntsm              │
│ Commit ID: 4e411c1a              │
│ Description: (no description)    │
│ Files: <any changes you make>   │
└─────────────────────────────────┘
         ↓ parent
┌─────────────────────────────────┐
│ Parent Commit                    │
│ Change ID: zzzzzzzz              │
│ Commit ID: 00000000              │
└─────────────────────────────────┘
```

### The `@` Symbol

- `@` always refers to the working copy commit
- It's your current "HEAD" in Git terms
- Can be used in any command that accepts a revision

```bash
jj show @          # Show current commit
jj log -r @-       # Show parent commit
jj rebase -d @-    # Rebase onto parent (undo last commit)
jj diff -r @^      # Diff against parent
```

### Working Copy Operations

```bash
# Switch working copy to different commit
jj edit <revision>

# Create new working copy (child of current)
jj new

# Create new working copy (child of specific revision)
jj new <revision>

# Abandon current working copy
jj abandon
```

## Deep Dive: Revsets

Revsets are expressions for selecting commits. They're extremely powerful.

### Basic Revsets

```bash
@                   # Current working copy
@-                  # Parent of working copy
@^                  # Parent of working copy (alternative syntax)
@--                 # Grandparent of working copy
<change-id>         # Specific change
main@origin         # Bookmark at remote
root()              # Root commit
```

### Revset Operators

```bash
# Range operators
x..y               # Ancestors of y that are not ancestors of x
x::y               # Commits from x to y (inclusive)
::x                # x and all its ancestors
x::                # x and all its descendants

# Set operators
x | y              # Union (either x or y)
x & y              # Intersection (both x and y)
~x                 # Negation (not x)

# Examples
main..@            # All commits from main to working copy
@- | @             # Parent and current commit
all() & ~mine()    # All commits not authored by me
```

### Revset Functions

```bash
# Authorship
author(pattern)         # Commits by author
committer(pattern)      # Commits by committer
mine()                  # Your own commits

# Relationships
ancestors(x)            # All ancestors of x
descendants(x)          # All descendants of x
parents(x)              # Direct parents
children(x)             # Direct children
heads(x)                # Heads in set x
roots(x)                # Roots in set x

# Branches/Bookmarks
bookmarks()             # All bookmarks
remote_bookmarks()      # All remote bookmarks
tags()                  # All tags

# Status
conflicted()            # Commits with conflicts
empty()                 # Empty commits
description(pattern)    # Commits matching description

# Examples
jj log -r 'ancestors(@) & author(alice)'
jj log -r 'descendants(main) & ~empty()'
jj log -r 'bookmarks() | remote_bookmarks()'
```

### Practical Revset Examples

```bash
# Show only your work since main
jj log -r 'main..@ & mine()'

# Find all merge commits
jj log -r 'heads(all()) & parents(all())'

# Show commits that need description
jj log -r 'description(exact:"")'

# Show recent commits excluding merge commits
jj log -r '@ & ~merge()'

# All commits that touch a specific file
jj log path/to/file
```

## Deep Dive: Bookmarks

Bookmarks are jj's version of Git branches, but less central to the workflow.

### Philosophy Difference

```
Git Workflow:              Jujutsu Workflow:
─────────────             ─────────────────
1. Create branch          1. Create change
2. Make commits           2. Make more changes
3. Push branch            3. Create bookmark
                          4. Push bookmark
↑ Branch-centric          ↑ Change-centric
```

### Bookmark Operations

```bash
# Create bookmark at current commit
jj bookmark create feature-x

# Create bookmark at specific commit
jj bookmark create feature-x -r <revision>

# Move bookmark to different commit
jj bookmark set feature-x -r <revision>

# List bookmarks
jj bookmark list

# List all bookmarks (including remotes)
jj bookmark list --all

# Delete bookmark
jj bookmark delete feature-x

# Rename bookmark
jj bookmark rename old-name new-name
```

### Tracking Remote Bookmarks

```bash
# Track a remote bookmark (like git branch --track)
jj bookmark track main@origin

# Untrack a remote bookmark
jj bookmark untrack main@origin

# List tracked bookmarks
jj bookmark list --tracked

# Forget a bookmark entirely (local + untrack remote)
jj bookmark forget feature-x
```

### Bookmark vs Git Branch

| Aspect | Git Branch | jj Bookmark |
|--------|-----------|-------------|
| Importance | Central to workflow | Optional, mainly for remotes |
| Creation | Before making commits | After creating changes |
| Moving | Automatically with commits | Explicitly moved |
| Checking out | Required to work | Not required |
| Deletion | Can lose commits | Just a pointer |

## Deep Dive: Operations Log

Every jj command is recorded in the operation log.

### Viewing Operations

```bash
# View operation history
jj op log

# Example output:
# @  a1b2c3d4 user@host 2025-10-15 10:23:45
# │  describe commit abc123
# ◉  e5f6g7h8 user@host 2025-10-15 10:20:12
# │  commit working copy
# ◉  i9j0k1l2 user@host 2025-10-15 10:15:03
#    new empty change

# Show specific operation details
jj op show <operation-id>
```

### Undoing Operations

```bash
# Undo last operation
jj undo

# Undo multiple operations
jj undo  # Once
jj undo  # Twice

# Restore to specific operation
jj op restore <operation-id>

# Diff between operations
jj op diff <operation-id1> <operation-id2>
```

### Operation Safety Net

The operation log is your safety net:
- Every operation is logged
- Can restore to any previous state
- Operations are garbage collected after 90 days (configurable)

```bash
# Example: Recover from accidental abandon
jj abandon @        # Oops! Didn't mean to do that
jj undo             # Phew! Restored
```

## Deep Dive: Conflict Resolution

### How Conflicts Work

```bash
# Git approach:
git merge feature
# CONFLICT!
# Must resolve now, working directory has conflict markers
# Can't commit, can't rebase, can't checkout

# Jujutsu approach:
jj rebase -d feature
# Rebase completes!
# Conflict is recorded in commit
# Can continue working, resolve later
```

### Conflict State

```bash
$ jj log
@  qpvuntsm (conflict) Merge feature
│  ↑ Conflict marker in log
◉  abc123de Add feature
│
◉  xyz789ab Main work

$ jj status
Working copy : qpvuntsm (conflict)
Conflicts:
  file.txt: 2-sided conflict

# File has conflict markers but you can:
# - Work on other changes
# - Resolve later
# - Push (if you really want to)
```

### Three-Way Merge Representation

```
     Base
     /  \
  Left  Right
     \  /
    Merged
```

Jujutsu stores all three versions internally and can regenerate conflict markers.

## Mental Model Summary

### Git Mental Model

```
[Working Directory] ← Dirty or clean
        ↓
[Staging Area] ← Carefully curated
        ↓
[Commits] ← Immutable history
        ↓
[Branches] ← Primary organization
        ↓
[Remote] ← Shared truth
```

### Jujutsu Mental Model

```
[Working Copy Commit] ← Always a commit, always clean
        ↓
[Change Stack] ← Mutable, freely editable
        ↓
[Bookmarks] ← Optional labels
        ↓
[Remote] ← Immutable shared truth
        ↓
[Operation Log] ← Safety net for everything
```

## Key Principles

1. **Everything is a commit** - Including your working directory
2. **Changes are mutable** - Until pushed to a remote
3. **Operations are logged** - Everything can be undone
4. **Conflicts are data** - Not blocking errors
5. **Automatic rebasing** - Descendants follow parents
6. **Bookmarks are optional** - Changes come first
7. **Git compatible** - Full interoperability maintained

## Common Misconceptions

### "jj doesn't have commits"
**False**: jj has commits. The difference is:
- Working copy IS a commit
- Commits are mutable until pushed
- Changes (logical units) vs commits (snapshots)

### "jj doesn't have branches"
**Partially true**: jj has bookmarks (similar to branches) but:
- Less central to the workflow
- Created after changes, not before
- Mainly used for pushing to remotes

### "jj is completely different from Git"
**False**: jj is Git-compatible:
- Uses Git repositories as backend
- Creates real Git commits
- Works with Git remotes
- Can use both tools on same repo

### "No staging area means less control"
**False**: You have more control:
- `jj split` for granular commits
- `jj squash -i` for interactive staging
- `jj absorb` for intelligent distribution
- Edit any commit at any time

## Next Steps

Now that you understand the core concepts:

1. [Commands Reference](Commands-Reference.md) - Detailed command documentation
2. [Workflows](Workflows.md) - Apply concepts in practice
3. [Advanced Features](Advanced-Features.md) - Leverage the power of jj

---

[← Back to Getting Started](Getting-Started.md) | [Main](README.md) | [Next: Commands Reference →](Commands-Reference.md)
