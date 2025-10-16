# Commands Reference

Comprehensive reference for all Jujutsu commands, organized by category.

## Quick Reference Table

| Category | Common Commands |
|----------|----------------|
| **Status & Info** | `status`, `log`, `show`, `diff` |
| **Changes** | `new`, `edit`, `describe`, `abandon` |
| **History** | `rebase`, `squash`, `split`, `duplicate` |
| **Bookmarks** | `bookmark create/list/delete`, `bookmark track` |
| **Remote** | `git fetch`, `git push`, `git clone` |
| **Conflicts** | `resolve`, `absorb` |
| **Safety** | `undo`, `op log`, `op restore` |

---

## Status & Information

### `jj status`

Show the working copy status.

```bash
# Basic status
jj status

# Example output:
# Working copy : qpvuntsm 4e411c1a
# Parent commit: zzzzzzzz 00000000
# Added files:
#   new-file.txt
# Modified files:
#   existing-file.txt
```

**Common flags:**
- None commonly used - it's simple by design

---

### `jj log`

Show commit history.

```bash
# Show recent history
jj log

# Show all history
jj log -r 'all()'

# Show specific range
jj log -r 'main..@'

# Show with full commit IDs
jj log --no-graph --color=always

# Limit number of commits
jj log -n 10

# Show specific path history
jj log path/to/file
```

**Common flags:**
- `-r, --revisions <REVSET>` - Revisions to show
- `-n, --limit <NUM>` - Limit number of commits
- `--no-graph` - Don't show the graph
- `-T, --template <TEMPLATE>` - Custom template
- `-p, --patch` - Show diff

**Examples:**
```bash
# Your commits since main
jj log -r 'main..@ & mine()'

# Last 5 commits
jj log -n 5

# Commits that touched a file
jj log src/main.rs

# With diffs
jj log -p -n 3
```

---

### `jj show`

Show details of a revision.

```bash
# Show current commit
jj show

# Show specific commit
jj show <revision>

# Show with full diff
jj show --git

# Show summary only
jj show --summary

# Show specific file
jj show <revision> path/to/file
```

**Examples:**
```bash
jj show @-          # Show parent commit
jj show main        # Show main bookmark
jj show --git       # Git-style diff format
```

---

### `jj diff`

Show changes between revisions.

```bash
# Show uncommitted changes
jj diff

# Show changes in specific commit
jj diff -r <revision>

# Diff between two commits
jj diff --from <rev1> --to <rev2>

# Show diff for specific files
jj diff path/to/file

# Show summary of changes
jj diff --summary

# Show stat summary
jj diff --stat
```

**Common flags:**
- `-r, --revision <REV>` - Show changes in this revision
- `--from <REV>` - Show changes from this revision
- `--to <REV>` - Show changes to this revision
- `--stat` - Show diffstat
- `--summary` - Show summary
- `--git` - Git-style diff

**Examples:**
```bash
# Changes in working copy
jj diff

# Changes in parent commit
jj diff -r @-

# Diff between main and current
jj diff --from main --to @

# See what changed in specific file
jj diff -r main src/main.rs
```

---

## Creating & Editing Changes

### `jj new`

Create a new change.

```bash
# Create new change on current commit
jj new

# Create with description
jj new -m "Add new feature"

# Create as child of specific commit
jj new <revision>

# Create from specific commit with message
jj new <revision> -m "Fix bug"

# Create multiple parallel changes
jj new <revision> -m "Change 1"
jj new <revision> -m "Change 2"

# Insert new change between commits
jj new --before <revision>
jj new --after <revision>
```

**Common flags:**
- `-m, --message <MSG>` - Set description
- `-r, --revision <REV>` - Create based on this revision
- `--before <REV>` - Insert before this revision
- `--after <REV>` - Insert after this revision

**Examples:**
```bash
# Start new work
jj new -m "Implement user auth"

# Create change based on main
jj new main -m "Feature branch"

# Insert change in the middle of history
jj new --after @-- -m "Forgot this change"
```

---

### `jj edit`

Set a revision as the working copy.

```bash
# Edit specific commit
jj edit <revision>

# Edit parent
jj edit @-

# Edit by change ID
jj edit qpvuntsm
```

**Use cases:**
- Modify a past commit
- Go back in history
- Fix something in the middle of a stack

**Examples:**
```bash
# Go back two commits
jj edit @--

# Edit a specific change
jj edit qpvuntsm

# Return to latest (if you know the change ID)
jj edit <latest-change-id>
```

**Tip**: Use `jj log` to find the change you want to edit.

---

### `jj describe`

Edit the description of a change.

```bash
# Describe current change
jj describe

# Describe with message inline
jj describe -m "New description"

# Describe specific change
jj describe -r <revision> -m "Description"

# Open editor for current change
jj describe

# Clear description
jj describe -m ""
```

**Common flags:**
- `-m, --message <MSG>` - Set description
- `-r, --revision <REV>` - Change to describe
- `--stdin` - Read description from stdin

**Examples:**
```bash
# Open editor to write description
jj describe

# Set description inline
jj describe -m "Fix login bug"

# Describe past commit
jj describe -r @- -m "Updated commit message"
```

---

### `jj abandon`

Abandon a change (remove it from history).

```bash
# Abandon current change
jj abandon

# Abandon specific change
jj abandon <revision>

# Abandon multiple changes
jj abandon <rev1> <rev2>

# Abandon range
jj abandon -r '<revset>'
```

**What happens:**
- Change is removed from history
- Children are rebased onto the parent
- Working copy files are NOT deleted

**Examples:**
```bash
# Abandon current change
jj abandon @

# Abandon a specific commit and its descendants
jj abandon <change-id>

# Abandon empty commits
jj abandon -r 'empty()'
```

---

## History Modification

### `jj squash`

Move changes from one revision into another.

```bash
# Squash working copy into parent
jj squash

# Squash with interactive selection
jj squash -i

# Squash into specific destination
jj squash --into <revision>

# Squash from specific revision
jj squash -r <revision>

# Squash specific files
jj squash path/to/file
```

**Common flags:**
- `-i, --interactive` - Interactive selection
- `-r, --revision <REV>` - Source revision (default: @)
- `--into <REV>` - Destination (default: parent of source)
- `-m, --message <MSG>` - Set description of destination

**Examples:**
```bash
# Squash all changes into parent
jj squash

# Interactively choose what to squash
jj squash -i

# Squash specific file into parent
jj squash src/file.rs

# Squash older commit into its parent
jj squash -r @-
```

---

### `jj split`

Split a change into two changes.

```bash
# Interactively split current change
jj split

# Split specific change
jj split -r <revision>

# Split with path filter
jj split path/to/file
```

**How it works:**
- Opens interactive editor
- Select which changes go in first commit
- Remaining changes go in second commit

**Examples:**
```bash
# Split working copy commit
jj split

# Split a past commit
jj split -r @-

# Split keeping specific file in first commit
jj split src/feature.rs
```

---

### `jj rebase`

Move revisions to a different parent.

```bash
# Rebase current change onto new parent
jj rebase -d <destination>

# Rebase specific change
jj rebase -r <revision> -d <destination>

# Rebase multiple changes
jj rebase -s <source> -d <destination>

# Rebase branch (all descendants)
jj rebase -b <branch> -d <destination>
```

**Common flags:**
- `-d, --destination <REV>` - New parent
- `-r, --revision <REV>` - Revision to rebase
- `-s, --source <REV>` - Rebase this and descendants
- `-b, --branch <REV>` - Rebase entire branch

**Examples:**
```bash
# Rebase onto main
jj rebase -d main

# Rebase older commit
jj rebase -r @-- -d main

# Rebase entire stack onto main
jj rebase -s @--- -d main
```

---

### `jj duplicate`

Create a copy of a revision.

```bash
# Duplicate current change
jj duplicate

# Duplicate specific change
jj duplicate <revision>

# Duplicate multiple changes
jj duplicate <rev1> <rev2>
```

**Use cases:**
- Experiment with different approaches
- Cherry-pick equivalent
- Create backup before risky operation

**Examples:**
```bash
# Duplicate current change
jj duplicate @

# Duplicate and edit the copy
jj duplicate @ && jj edit <new-change-id>

# Duplicate a specific commit
jj duplicate main
```

---

### `jj absorb`

Automatically distribute changes to appropriate ancestors.

```bash
# Absorb working copy changes
jj absorb

# Absorb from specific revision
jj absorb -r <revision>
```

**How it works:**
- Analyzes which lines were modified in ancestors
- Moves each change to the most recent ancestor that modified those lines
- Like an intelligent `git commit --fixup`

**Examples:**
```bash
# Make small fixes across multiple files
# Edit files...
jj absorb  # Automatically puts fixes in right commits!
```

---

### `jj move`

Move changes from one revision to another.

```bash
# Move changes from source to destination
jj move --from <rev> --to <rev>

# Interactively move changes
jj move --from <rev> --to <rev> -i
```

---

## Bookmarks (Branches)

### `jj bookmark create`

Create a new bookmark.

```bash
# Create bookmark at current commit
jj bookmark create feature-x

# Create at specific commit
jj bookmark create feature-x -r <revision>
```

**Examples:**
```bash
jj bookmark create my-feature
jj bookmark create hotfix -r @-
```

---

### `jj bookmark list`

List bookmarks.

```bash
# List local bookmarks
jj bookmark list

# List all (including remote) bookmarks
jj bookmark list --all

# List tracked bookmarks
jj bookmark list --tracked

# List conflicted bookmarks
jj bookmark list --conflicted
```

**Examples:**
```bash
jj bookmark list           # Local only
jj bookmark list --all     # Include remotes
jj bookmark list -t        # Only tracked
```

---

### `jj bookmark delete`

Delete a bookmark.

```bash
# Delete local bookmark
jj bookmark delete feature-x

# Delete multiple bookmarks
jj bookmark delete feature-1 feature-2

# Delete with glob pattern
jj bookmark delete 'glob:feature-*'
```

**Examples:**
```bash
jj bookmark delete old-feature
jj bookmark delete 'glob:temp-*'
```

---

### `jj bookmark set`

Move a bookmark to a different commit.

```bash
# Move bookmark to current commit
jj bookmark set feature-x

# Move to specific commit
jj bookmark set feature-x -r <revision>

# Allow moving backward (discouraged)
jj bookmark set feature-x -r <earlier-commit> --allow-backwards
```

---

### `jj bookmark track`

Track a remote bookmark.

```bash
# Track remote bookmark
jj bookmark track main@origin

# Track all remote bookmarks
jj bookmark track 'glob:*@origin'
```

---

### `jj bookmark untrack`

Stop tracking a remote bookmark.

```bash
# Untrack specific bookmark
jj bookmark untrack main@origin

# Untrack all from remote
jj bookmark untrack 'glob:*@origin'
```

---

### `jj bookmark forget`

Delete a bookmark and untrack remotes.

```bash
# Forget bookmark entirely
jj bookmark forget feature-x
```

---

## Git Operations

### `jj git clone`

Clone a Git repository.

```bash
# Clone in colocated mode (recommended)
jj git clone --colocate https://github.com/user/repo.git

# Clone without colocation
jj git clone https://github.com/user/repo.git

# Clone to specific directory
jj git clone --colocate https://github.com/user/repo.git my-dir

# Clone with SSH
jj git clone --colocate git@github.com:user/repo.git
```

---

### `jj git init`

Initialize a Git-backed repository.

```bash
# Initialize in current directory (colocated)
jj git init --colocate

# Initialize new directory
jj git init --colocate my-project

# Initialize without colocation
jj git init

# Initialize from existing Git repo
jj git init --git-repo=../existing-repo
```

---

### `jj git fetch`

Fetch from Git remotes.

```bash
# Fetch from default remote
jj git fetch

# Fetch from specific remote
jj git fetch --remote origin

# Fetch from all remotes
jj git fetch --all-remotes

# Fetch specific branch
jj git fetch --branch main
```

**Common flags:**
- `--remote <NAME>` - Specific remote
- `--all-remotes` - All remotes
- `--branch <NAME>` - Specific branch

---

### `jj git push`

Push to Git remotes.

```bash
# Push current bookmark
jj git push

# Push specific bookmark
jj git push --bookmark feature-x

# Push all bookmarks
jj git push --all

# Push to specific remote
jj git push --remote upstream

# Create bookmark and push
jj bookmark create feature-x
jj git push --bookmark feature-x

# Force push (use with caution)
jj git push --bookmark feature-x --force

# Dry run
jj git push --bookmark feature-x --dry-run
```

**Common flags:**
- `--bookmark <NAME>` - Push specific bookmark
- `--all` - Push all bookmarks
- `--remote <NAME>` - Push to specific remote
- `--force` - Force push
- `--dry-run` - Show what would be pushed

---

### `jj git remote`

Manage Git remotes.

```bash
# List remotes
jj git remote list

# Add remote
jj git remote add origin https://github.com/user/repo.git

# Remove remote
jj git remote remove origin

# Rename remote
jj git remote rename origin upstream
```

---

## Conflict Resolution

### `jj resolve`

Resolve conflicts.

```bash
# Resolve conflicts in current change
jj resolve

# List conflicted files
jj resolve --list

# Resolve specific file
jj resolve path/to/file

# Use tool for specific file
jj resolve --tool meld path/to/file
```

**Common flags:**
- `--list` - List conflicted files
- `--tool <TOOL>` - Use specific merge tool

**Examples:**
```bash
# List conflicts
jj resolve --list

# Resolve interactively
jj resolve

# Use specific tool
jj resolve --tool vimdiff src/main.rs
```

---

## File Operations

### `jj file list`

List files in a revision.

```bash
# List files in working copy
jj file list

# List files in specific revision
jj file list -r <revision>

# List with paths
jj file list -r @
```

---

### `jj file show`

Show file contents.

```bash
# Show file from current commit
jj file show path/to/file

# Show from specific revision
jj file show -r <revision> path/to/file
```

---

### `jj file track`

Start tracking paths.

```bash
# Track specific path
jj file track path/to/file

# Track pattern
jj file track 'glob:src/**/*.rs'
```

---

## Safety & Recovery

### `jj undo`

Undo the last operation.

```bash
# Undo last operation
jj undo

# Undo multiple times
jj undo
jj undo
```

**What gets undone:**
- Last jj command that modified the repo
- Includes commits, rebases, squashes, etc.

---

### `jj op log`

Show operation history.

```bash
# Show operation log
jj op log

# Show more operations
jj op log -n 50

# Show with timestamps
jj op log --timestamp
```

---

### `jj op restore`

Restore to a specific operation.

```bash
# Restore to specific operation
jj op restore <operation-id>

# Example:
jj op log  # Find the operation ID
jj op restore a1b2c3d4
```

---

### `jj op show`

Show details of an operation.

```bash
# Show operation details
jj op show <operation-id>
```

---

## Configuration

### `jj config`

Manage configuration.

```bash
# Edit user config
jj config edit --user

# Edit repo config
jj config edit --repo

# Set config value
jj config set --user user.name "Your Name"
jj config set --user user.email "you@example.com"

# Get config value
jj config get user.name

# List all config
jj config list
```

---

## Utility Commands

### `jj workspace`

Manage workspaces (working copies).

```bash
# List workspaces
jj workspace list

# Add new workspace
jj workspace add ../other-workspace

# Remove workspace
jj workspace forget <name>
```

---

### `jj util completion`

Generate shell completion scripts.

```bash
# Generate bash completion
jj util completion bash

# Generate zsh completion
jj util completion zsh

# Generate fish completion
jj util completion fish

# Generate PowerShell completion
jj util completion powershell
```

---

## Advanced Commands

### `jj commit`

Update the description and create a new working copy.

```bash
# Like git commit
jj commit -m "Commit message"

# Equivalent to:
jj describe -m "Commit message"
jj new
```

---

### `jj next` / `jj prev`

Navigate through history.

```bash
# Move to next (child) commit
jj next

# Move to previous (parent) commit
jj prev

# Move multiple steps
jj next --edit
jj prev --edit
```

---

### `jj diffedit`

Edit diff interactively.

```bash
# Interactively edit changes
jj diffedit
```

---

## Command Chaining

Common patterns for combining commands:

```bash
# Create, edit, and push
jj new -m "Feature" && \
  # edit files && \
  jj bookmark create feature && \
  jj git push --bookmark feature

# Squash and continue
jj squash && jj new -m "Next change"

# Edit, amend, return
jj edit @- && \
  # make changes && \
  jj edit @  # return to tip
```

---

## Global Options

These work with any command:

```bash
--help, -h              # Show help
--repository <PATH>     # Repository path
--color <WHEN>          # When to use color (always, never, auto)
--config-toml <TOML>    # Additional config
--ignore-working-copy   # Don't snapshot working copy
```

---

[← Back to Core Concepts](Core-Concepts.md) | [Main](README.md) | [Next: Git Comparison →](Git-Comparison.md)
