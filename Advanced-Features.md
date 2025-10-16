# Advanced Features

Powerful Jujutsu features for expert users and complex workflows.

## Advanced Change Manipulation

### `jj absorb` - Intelligent Distribution

Automatically distribute changes to the commits that last modified those lines.

```bash
# You have a stack of commits
jj log
# @  Working copy (has fixes)
# ◉  C  Add feature
# ◉  B  Add tests
# ◉  A  Add structure

# Make fixes to files modified in A, B, and C
echo "fix for A" >> file-a.txt
echo "fix for B" >> file-b.txt
echo "fix for C" >> file-c.txt

# Absorb automatically puts each fix in the right commit!
jj absorb

# Result:
jj log
# @  Working copy (empty)
# ◉  C'  Add feature (with fix)
# ◉  B'  Add tests (with fix)
# ◉  A'  Add structure (with fix)
```

**How it works:**
- Analyzes which commits last modified each line
- Moves changes to the most recent modifier
- Preserves logical organization

**Use cases:**
- Fixing typos across multiple commits
- Updating after code review
- Cleaning up after refactoring

### `jj split` - Interactive Commit Splitting

Break a large commit into smaller, logical pieces.

```bash
# You have a large commit
jj log
# @  Big commit with many unrelated changes

# Split interactively
jj split

# Built-in editor opens showing all changes
# Select which changes go in first commit:
#   [x] file1.txt
#   [x] file2.txt
#   [ ] file3.txt
#   [ ] file4.txt
# Press 'c' to confirm

# Result: Two commits
jj log
# @  Changes to file3.txt and file4.txt
# ◉  Changes to file1.txt and file2.txt

# Describe each properly
jj describe -m "Add core functionality"
jj edit @-
jj describe -m "Add supporting files"
```

**Advanced patterns:**
```bash
# Split by path
jj split src/  # Split keeping only src/ changes in first commit

# Split specific files
jj split file1.txt file2.txt  # These go in first commit
```

### `jj squash` - Flexible Squashing

Move changes between commits with fine control.

```bash
# Basic squash (into parent)
jj squash

# Interactive squash (choose what to squash)
jj squash -i

# Squash into specific commit
jj squash --into <revision>

# Squash specific file
jj squash src/main.rs

# Squash from different commit
jj squash -r <source> --into <dest>
```

**Advanced patterns:**
```bash
# Squash all but one file
jj squash --into @- --from @ --except config.toml

# Squash with custom message
jj squash -m "Combined changes"
```

### `jj move` - Surgical Change Moving

Move specific changes between commits.

```bash
# Move changes from one commit to another
jj move --from <source> --to <dest>

# Interactive selection
jj move --from @ --to @- -i

# Move specific paths
jj move --from @ --to @- src/feature.rs
```

## Advanced Rebasing

### Rebasing Ranges

```bash
# Rebase a single commit
jj rebase -r <commit> -d <dest>

# Rebase commit and all descendants
jj rebase -s <commit> -d <dest>

# Rebase entire branch
jj rebase -b <branch-head> -d <dest>
```

### Insert Commits in History

```bash
# Insert new commit between existing commits
jj new --after @-- -m "Forgot this"

# Visual:
# Before:           After:
# @  C              @  C (rebased)
# ◉  B              ◉  New commit
# ◉  A              ◉  B (rebased)
#                   ◉  A

# Or insert before
jj new --before @ -m "Needs to come first"
```

### Complex Rebase Scenarios

```bash
# Rebase onto multiple parents (create merge)
jj rebase -d 'parent1 | parent2'

# Rebase with revset
jj rebase -s 'mine() & ~ancestors(main@origin)' -d main@origin

# Rebase preserving merge commits
jj rebase -r <merge-commit> -d <new-base>
```

## Advanced Revsets

### Powerful Revset Queries

```bash
# All your commits not in main
jj log -r 'mine() & ~ancestors(main@origin)'

# Commits that touch specific file
jj log -r 'file("src/main.rs")'

# Empty commits
jj log -r 'empty()'

# Commits with conflicts
jj log -r 'conflicted()'

# Commits without description
jj log -r 'description(exact:"")'

# Merge commits (multiple parents)
jj log -r 'merges()'

# Recent commits by date
jj log -r 'ancestors(all(), 10)'

# Commits between two bookmarks
jj log -r 'bookmark-a::bookmark-b'

# Commits reachable from HEAD but not main
jj log -r '@ & ~main@origin'
```

### Combining Revsets

```bash
# Union (OR)
jj log -r 'mine() | conflicted()'

# Intersection (AND)
jj log -r 'mine() & conflicted()'

# Difference (NOT)
jj log -r 'all() & ~mine()'

# Complex queries
jj log -r '(mine() | author("colleague")) & ~ancestors(main@origin) & ~empty()'
```

### Revset Functions

```bash
# Ancestors and descendants
jj log -r 'ancestors(@, 5)'      # 5 generations back
jj log -r 'descendants(main)'    # All commits after main
jj log -r 'ancestors(@) & descendants(main)'  # Between main and @

# Parents and children
jj log -r 'parents(@)'           # Direct parents
jj log -r 'children(main)'       # Direct children

# Heads and roots
jj log -r 'heads(all())'         # All branch heads
jj log -r 'roots(mine())'        # Roots of your commits

# Bookmarks
jj log -r 'bookmarks()'          # All bookmarks
jj log -r 'remote_bookmarks()'   # All remote bookmarks
jj log -r 'bookmarks() & mine()' # Your bookmarks
```

## Operation Log Mastery

### Advanced Operation Commands

```bash
# View detailed operation log
jj op log -n 50

# Show specific operation
jj op show <operation-id>

# Diff between operations
jj op diff <op1> <op2>

# Restore to specific operation
jj op restore <operation-id>

# Undo multiple operations
jj undo && jj undo && jj undo

# View operation at specific time
jj op log --at '2 hours ago'
```

### Operation Log Workflows

#### Recover Lost Work

```bash
# You abandoned something important
jj abandon @

# Find it in operation log
jj op log
# Look for the abandon operation

# Undo that operation
jj undo

# Or restore to before that operation
jj op restore <operation-before-abandon>
```

#### Experiment and Rollback

```bash
# Try risky refactoring
jj op log  # Note current operation

# Do risky changes...
jj rebase -s <big-stack> -d <new-place>
jj squash -r <many-commits>
# Tests fail!

# Rollback everything
jj op restore <operation-before-experiment>

# Try different approach
```

#### Sync Across Machines

```bash
# Machine 1: Note operation
jj op log
# Latest: a1b2c3d4

# Machine 2: Restore to same operation
jj git fetch
jj op restore a1b2c3d4
```

## Duplicate and Duplicate

### Creating Commit Copies

```bash
# Duplicate current commit
jj duplicate @

# Duplicate specific commit
jj duplicate <revision>

# Duplicate multiple commits
jj duplicate <rev1> <rev2> <rev3>

# Duplicate range
jj duplicate -r 'main..@'
```

### Use Cases

#### Experiment with Different Approaches

```bash
# Create backup
jj duplicate @

# Experiment on original
# Try approach 1...

# Compare
jj diff -r @ --to <duplicate-id>

# Keep better one
jj abandon <worse-one>
```

#### Cherry-Pick Equivalent

```bash
# Want to copy commit from another branch
jj duplicate <commit-from-other-branch>
jj rebase -r <duplicate-id> -d @

# Now you have that commit in your branch
```

## Multiple Workspaces

Work on multiple changes simultaneously in separate directories.

### Creating Workspaces

```bash
# From main repo
jj workspace add ../my-feature

# Now you have two directories:
# ./         - Original workspace
# ../my-feature - New workspace

# Both share the same repo!
cd ../my-feature
jj log  # Same history
```

### Using Multiple Workspaces

```bash
# Original workspace: work on feature A
cd ~/repo
jj new -m "Feature A"
# Edit files...

# New workspace: work on feature B (simultaneously!)
cd ~/repo-feature-b
jj new -m "Feature B"
# Edit files...

# Both changes are tracked
jj log  # Shows both
```

### Managing Workspaces

```bash
# List workspaces
jj workspace list

# Forget workspace (remove reference, not files)
jj workspace forget feature-workspace

# Clean up workspace directory manually
rm -rf ../feature-workspace
```

## Advanced Conflict Handling

### Committing Conflicts

In jj, you can actually commit conflicts!

```bash
# Rebase causes conflict
jj rebase -d main@origin

jj log
# @  (conflict) My feature

# Commit it anyway (if you want to push for help)
jj bookmark create needs-help
jj git push --bookmark needs-help --force

# Others can see the conflict and help
# Or you can resolve later
```

### Multi-way Merges

```bash
# Merge multiple branches
jj new <branch1> <branch2> <branch3>

# Creates commit with three parents
# Resolve conflicts considering all three
```

### Conflict Resolution with Scripts

```bash
# Auto-resolve with formatter
jj resolve --tool ':builtin'
# Then run formatter
jj diff | rustfmt

# Or create custom resolution tool
jj config set --user merge-tools.custom-resolve.program "~/bin/auto-resolve.sh"
jj resolve --tool custom-resolve
```

## Custom Templates

### Advanced Template Syntax

```toml
[templates]
# Rich log format
log = '''
concat(
  label("change_id", change_id.shortest(8)),
  " ",
  label("commit_id", commit_id.shortest(8)),
  " ",
  if(conflict, label("conflict", "×"), " "),
  if(empty, label("empty", "○"), "●"),
  " ",
  separate(" | ",
    bookmarks.map(|b| label("bookmark", b.name())).join(", "),
    tags.map(|t| label("tag", t.name())).join(", "),
  ),
  "\n  ",
  description.first_line(),
  "\n  ",
  label("author", author.name()),
  " · ",
  label("timestamp", timestamp.ago()),
)
'''
```

### Conditional Formatting

```toml
[template-aliases]
# Show different icon based on state
'status_icon(change)' = '''
if(change.conflict, "×",
  if(change.empty, "○",
    if(change.working_copy, "●",
      "◆")))
'''

# Custom bookmark display
'show_bookmarks(bookmarks)' = '''
if(bookmarks,
  concat(" [", bookmarks.map(|b| b.name()).join(", "), "]"),
  "")
'''
```

## Advanced Git Integration

### Working with Multiple Remotes

```bash
# Add multiple remotes
jj git remote add upstream https://github.com/original/repo.git
jj git remote add origin https://github.com/you/repo.git

# Fetch from specific remote
jj git fetch --remote upstream

# Push to specific remote
jj git push --remote origin --bookmark feature

# Track bookmarks from different remotes
jj bookmark track main@upstream
jj bookmark track main@origin
```

### Git Submodules (Limited Support)

```bash
# Submodules are visible but not fully managed
jj file list  # Shows submodule directories

# Use git for submodule operations
git submodule update --init --recursive
jj status  # jj sees the changes
```

### Advanced Push Strategies

```bash
# Push multiple bookmarks
jj git push --bookmark feature-1 --bookmark feature-2

# Push with different names
jj bookmark create local-name
jj git push --bookmark local-name:remote-name

# Push all local bookmarks
jj git push --all

# Dry run to see what would be pushed
jj git push --all --dry-run
```

## Performance Optimization

### Large Repositories

```bash
# Use non-colocated mode for better performance
jj git clone https://github.com/large/repo.git  # No --colocate

# Or shallow clone
git clone --depth 1 https://github.com/large/repo.git
cd repo
jj git init --colocate
```

### Sparse Checkouts

```bash
# jj doesn't directly support sparse checkout
# But you can use git:
git sparse-checkout init
git sparse-checkout set src/
jj status  # Works with sparse checkout
```

### Operation Log Cleanup

```bash
# Operations older than 90 days are auto-cleaned
# Configure differently:
jj config set --user operation.gc-after-days 180

# Manual cleanup (rarely needed)
jj op abandon --ages '> 90 days'
```

## Scripting and Automation

### Using jj in Scripts

```bash
#!/bin/bash
# auto-sync.sh

# Fetch latest
jj git fetch --all-remotes

# Rebase current work
jj rebase -d main@origin

# Check for conflicts
if jj status | grep -q "conflict"; then
    echo "Conflicts detected!"
    exit 1
fi

# Run tests
./run-tests.sh

# Push if tests pass
jj git push --bookmark feature
```

### Custom Commands via Aliases

```toml
[aliases]
# Complete workflow
publish = ["!sh", "-c", "jj bookmark create $1 && jj git push --bookmark $1"]

# With environment
test-current = ["!cargo", "test"]

# Complex pipelines
auto-fixup = [
    "!sh", "-c",
    "jj absorb && jj squash"
]
```

### Parsing jj Output

```bash
# Get current change ID
CHANGE_ID=$(jj log -r @ --no-graph -T 'change_id.short()' | head -n1)

# Get list of modified files
FILES=$(jj status --no-pager | grep "Modified" | awk '{print $2}')

# Check for conflicts programmatically
if jj log -r @ -T 'conflict' | grep -q "true"; then
    echo "Current change has conflicts"
fi
```

## Debugging and Introspection

### Verbose Output

```bash
# Debug logging
JJ_LOG=debug jj log

# Trace all operations
JJ_LOG=trace jj rebase -d main

# Output to file
JJ_LOG=debug jj log 2> debug.log
```

### Inspecting Internal State

```bash
# View full commit object
jj show -r @ --git

# View operation details
jj op show <operation-id>

# Export to Git
jj git export  # Updates Git refs from jj
```

## Best Practices for Advanced Usage

### DO

✓ Use `jj absorb` for distributing fixes
✓ Split large commits before reviewing
✓ Leverage operation log for experimentation
✓ Create custom revset aliases for common queries
✓ Use workspaces for parallel feature development
✓ Write scripts to automate workflows

### DON'T

✗ Over-complicate with too many aliases
✗ Forget to test after `jj absorb`
✗ Rely solely on operation log as backup
✗ Create too many workspaces (confusing)
✗ Modify immutable history (after pushing)

## Next Steps

- [Builtin Editor](Builtin-Editor.md) - Master interactive editing
- [Best Practices](Best-Practices.md) - Apply advanced features effectively
- [Configuration](Configuration.md) - Customize for advanced workflows

---

[← Back to Configuration](Configuration.md) | [Main](README.md) | [Next: Builtin Editor →](Builtin-Editor.md)
