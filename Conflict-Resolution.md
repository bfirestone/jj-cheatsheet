# Conflict Resolution

How Jujutsu handles merge conflicts and practical strategies for resolving them.

## Jujutsu's Philosophy on Conflicts

### Key Difference from Git

| Git | Jujutsu |
|-----|---------|
| Conflicts **block** operations | Conflicts are **first-class** objects |
| Must resolve before continuing | Can resolve whenever ready |
| Working tree in inconsistent state | Conflicts recorded in commits |
| `git rebase --continue/--abort` | Just keep working or `jj undo` |

### What This Means

```bash
# Git approach:
git rebase main
# CONFLICT!
# ❌ Can't commit
# ❌ Can't switch branches
# ❌ Can't rebase other changes
# Must resolve NOW or abort

# Jujutsu approach:
jj rebase -d main
# Rebase completes!
# ✓ Can commit new changes
# ✓ Can switch to other changes
# ✓ Can rebase other things
# Resolve when ready
```

## Understanding Conflicts in jj

### Conflict Markers

Conflicts are visible in your log:

```bash
$ jj log
@  qpvuntsm (conflict) Merge feature
│  ↑ This indicates a conflict
◉  abc123de Add feature
│
◉  xyz789ab Main work
```

### Conflict State

```bash
$ jj status
Working copy : qpvuntsm (conflict)
There are unresolved conflicts at these paths:
  src/main.rs    2-sided conflict
  src/lib.rs     2-sided conflict
```

### Conflict in Files

Files with conflicts have markers:

```rust
<<<<<<< Conflict 1 of 1
%%%%%%% Changes from base to side #1
-old line
+new line from side 1
+++++++ Contents of side #2
new line from side 2
>>>>>>> Conflict 1 of 1 ends
```

## Basic Conflict Resolution

### Method 1: Manual Resolution

```bash
# 1. See which files have conflicts
jj status

# 2. Edit files to resolve
$EDITOR src/main.rs
# Remove conflict markers
# Choose correct resolution

# 3. Check resolution
jj diff

# 4. Done! Changes are automatically incorporated
jj status  # Should show no conflicts
```

### Method 2: Using `jj resolve`

```bash
# List conflicts
jj resolve --list

# Resolve interactively
jj resolve

# Resolve specific file
jj resolve src/main.rs

# Use external tool
jj resolve --tool meld src/main.rs
```

### Method 3: Use Built-in Editor

```bash
# Resolve with built-in editor
jj resolve

# Interactive TUI opens
# Navigate with arrow keys
# Select resolution
# Press 'c' to confirm
```

## Advanced Conflict Resolution

### Three-Way Merge Understanding

Every conflict has three sides:

```
      Base (common ancestor)
       /              \
   Left (your changes)  Right (their changes)
       \              /
        Merged result
```

Example:
```bash
# Base version:
fn hello() {
    println!("Hello");
}

# Your change (left):
fn hello() {
    println!("Hello, World!");
}

# Their change (right):
fn hello() {
    println!("Greetings");
}

# Conflict:
fn hello() {
<<<<<<< Conflict
%%%%%%% Changes from base to side #1
-    println!("Hello");
+    println!("Hello, World!");
+++++++ Contents of side #2
    println!("Greetings");
>>>>>>> Conflict ends
}
```

### Resolving Complex Conflicts

#### Strategy 1: Take One Side

```bash
# Take all of "ours" (current changes)
jj resolve --tool :local src/main.rs

# Take all of "theirs" (incoming changes)
jj resolve --tool :other src/main.rs

# Take base version
jj resolve --tool :base src/main.rs
```

#### Strategy 2: Resolve by Section

For large conflicts, resolve section by section:

```bash
# Edit file manually
$EDITOR src/main.rs

# For each conflict:
# 1. Understand both sides
# 2. Choose or combine
# 3. Remove markers

# Example resolution:
fn hello() {
    // Combined both changes
    println!("Greetings, World!");
}
```

#### Strategy 3: Use Merge Tool

```bash
# Configure merge tool
jj config set --user merge-tools.meld.program "meld"
jj config set --user ui.merge-editor "meld"

# Resolve with tool
jj resolve --tool meld src/main.rs

# Tool shows three panes:
# Left | Base | Right
# Edit middle pane to resolve
```

## Conflict Workflows

### Workflow 1: Resolve Immediately

```bash
# Create conflict
jj rebase -d main

jj log
# @  (conflict) My feature

# Resolve right away
jj resolve

# Edit conflicts...
# Save and exit

jj status
# Clean! No conflicts
```

### Workflow 2: Resolve Later

```bash
# Create conflict
jj rebase -d main

jj log
# @  (conflict) Feature

# Work on something else first
jj new main -m "Different work"
# Do other work...

# Come back to conflicts later
jj edit <conflict-change-id>
jj resolve
# Resolve now
```

### Workflow 3: Resolve Incrementally (Stack)

```bash
# Rebase stack with conflicts
jj rebase -s <bottom-of-stack> -d main

jj log
# @  C (conflict)
# ◉  B (conflict)
# ◉  A (no conflict)
# ◉  main

# Resolve bottom-up
jj edit <A-id>
# No conflict, skip

jj edit <B-id>
jj resolve
# Resolve B

# Note: C might auto-resolve now!
jj edit <C-id>
jj status
# Might be clean, or:
jj resolve
# Resolve C if needed
```

### Workflow 4: Resolve with Absorb

```bash
# Have conflict in merge
jj log
# @  (conflict) Merge

# Try auto-resolution
jj new
# Make manual fixes to files
# Edit conflicted sections...

# Absorb fixes into appropriate commits
jj absorb

# Check if conflicts resolved
jj log
# Might show no conflicts now!
```

## Dealing with Specific Conflict Types

### Binary File Conflicts

```bash
# Binary files can't be merged
jj status
# image.png: 2-sided conflict

# Choose one version:
# Keep ours:
cp image.png.left image.png

# Keep theirs:
cp image.png.right image.png

# Or use resolve with --tool
jj resolve --tool :local image.png
```

### Conflict in Deleted Files

```bash
# File deleted in one branch, modified in another
jj status
# deleted_file.rs: modify/delete conflict

# Keep deletion:
rm deleted_file.rs

# Keep modification:
# File already exists with changes
# Just leave it

# Status updates automatically
jj status
```

### Renamed File Conflicts

```bash
# File renamed differently in each branch
jj status
# file.rs: rename conflict
#   renamed to: file_a.rs (side 1)
#   renamed to: file_b.rs (side 2)

# Choose one name:
mv file_a.rs final_name.rs
rm file_b.rs

# Or merge contents:
cat file_a.rs file_b.rs > final_name.rs
```

## Merge Commits

### Creating Merge Commits

```bash
# Merge two changes
jj merge <left-change> <right-change>

# Example
jj merge feature-a feature-b

# Creates merge commit with both parents
jj log
# @  (empty) Merge
# ├─◉  feature-b
# │ │
# ◉ │  feature-a
# ├─╯
```

### Resolving Merge Conflicts

```bash
# Create merge (might have conflicts)
jj merge feature-a feature-b

jj status
# Working copy: (conflict)

# Resolve
jj resolve

# Now merge is clean
jj describe -m "Merge feature-a and feature-b"
```

## Conflict Resolution Tools

### Built-in Tools

```bash
# Built-in scm-record editor
jj resolve

# Features:
# - Interactive TUI
# - Side-by-side view
# - Syntax highlighting
# - Keyboard navigation
```

### External Tools

#### Meld

```bash
# Configure
jj config set --user merge-tools.meld.program "meld"

# Use
jj resolve --tool meld file.rs
```

#### Vimdiff

```bash
# Configure
jj config set --user merge-tools.vimdiff.program "vim"
jj config set --user merge-tools.vimdiff.merge-args = ["-d", "$left", "$base", "$right", "$output"]

# Use
jj resolve --tool vimdiff file.rs
```

#### VS Code

```bash
# Configure
jj config set --user merge-tools.vscode.program "code"
jj config set --user merge-tools.vscode.merge-args = ["--wait", "--merge", "$left", "$right", "$base", "$output"]

# Use
jj resolve --tool vscode file.rs
```

#### Difftastic

```bash
# For viewing diffs (not merging)
jj config set --user ui.diff.tool "difftastic"

# View conflict with better diff
jj diff
```

### Configuring Default Tool

```toml
# In ~/.config/jj/config.toml
[ui]
merge-editor = "meld"  # or vimdiff, vscode, etc.

[merge-tools.meld]
program = "meld"
```

## Preventing Conflicts

### Strategy 1: Rebase Frequently

```bash
# Stay up to date with main
jj git fetch
jj rebase -d main@origin

# Do this daily to minimize conflicts
```

### Strategy 2: Small, Focused Changes

```bash
# Instead of:
jj new -m "Huge refactoring touching everything"

# Do:
jj new -m "Refactor module A"
jj new -m "Refactor module B"
jj new -m "Refactor module C"

# Smaller changes = fewer conflicts
```

### Strategy 3: Communicate with Team

```bash
# Before big refactoring:
# 1. Announce in team chat
# 2. Coordinate timing
# 3. Have others merge/rebase first
# 4. Then do refactoring
# 5. Others rebase after
```

### Strategy 4: Use `jj absorb`

```bash
# Instead of creating conflicts:
# Make small fixes across commits
jj absorb  # Distributes to right places

# Reduces conflicts later
```

## Debugging Conflicts

### Show Conflict Details

```bash
# See detailed conflict info
jj show

# See which files have conflicts
jj status

# See diff including conflict markers
jj diff
```

### Understand Conflict Source

```bash
# Log to see what caused conflict
jj log -r '::@'

# See parents of conflict
jj show -r 'parents(@)'

# Diff each side
jj diff --from 'parents(@)[0]' --to @
jj diff --from 'parents(@)[1]' --to @
```

### Test Resolution

```bash
# After resolving
jj status  # Should show no conflicts

# Run tests
./run-tests.sh

# If tests fail, continue editing
# Changes automatically update the commit
```

## When Things Go Wrong

### Conflict Too Hard to Resolve

```bash
# Option 1: Undo the operation
jj undo

# Try different approach (rebase one commit at a time)
jj rebase -r <single-commit> -d main
jj resolve
jj rebase -r <next-commit> -d @
jj resolve
# Repeat...

# Option 2: Duplicate and try different strategy
jj duplicate @
# Try resolution on duplicate
# Keep whichever works
```

### Made Mistakes Resolving

```bash
# Undo and try again
jj undo
jj resolve  # Try again

# Or edit files more
# Changes automatically update
```

### Conflicts After Push

```bash
# Resolve locally
jj resolve

# Force push (if your branch)
jj git push --bookmark feature --force

# Or create new commit if shared
jj new -m "Resolve conflicts"
jj resolve
jj git push --bookmark feature
```

## Best Practices

### DO

✓ Resolve conflicts when you have context
✓ Test after resolving
✓ Resolve incrementally (bottom of stack first)
✓ Use `jj undo` if resolution goes wrong
✓ Commit with clear resolution description
✓ Use merge tools for complex conflicts
✓ Keep conflict markers clear and readable

### DON'T

✗ Rush conflict resolution
✗ Resolve without understanding both sides
✗ Ignore test failures after resolution
✗ Leave conflict markers in code
✗ Mix resolution with other changes
✗ Force-push to shared branches without communication

## Common Patterns

### Pattern 1: Conflict Storm

```bash
# Multiple conflicts after rebase
jj log
# @ D (conflict)
# ◉ C (conflict)
# ◉ B (conflict)
# ◉ A (no conflict)

# Strategy: Resolve bottom-up
for change in A B C D; do
    jj edit $change
    jj resolve
done
```

### Pattern 2: Trivial Conflicts

```bash
# Conflict is just whitespace or formatting
jj diff  # See conflict

# Auto-format might fix it
jj restore .  # Get base version
# Run formatter
./format-code.sh

# Or use jj fix (if configured)
jj fix
```

### Pattern 3: Take Theirs

```bash
# Conflict where their changes are correct
jj resolve --tool :other file.rs

# Or for all files:
for file in $(jj resolve --list | awk '{print $1}'); do
    jj resolve --tool :other $file
done
```

## Summary

| Aspect | Git | Jujutsu |
|--------|-----|---------|
| **Conflicts block** | Yes | No |
| **Resolution required** | Before continuing | Whenever ready |
| **Tools** | git mergetool | jj resolve |
| **Undo** | Hard | Easy (jj undo) |
| **View** | Working tree | First-class commit |
| **Workflow** | Interrupted | Continuous |

## Next Steps

- [Advanced Features](Advanced-Features.md) - More powerful techniques
- [Configuration](Configuration.md) - Configure merge tools
- [Best Practices](Best-Practices.md) - Modern conflict strategies

---

[← Back to Workflows](Workflows.md) | [Main](README.md) | [Next: Configuration →](Configuration.md)
