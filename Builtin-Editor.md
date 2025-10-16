# Built-in Diff/Merge Editor

Master Jujutsu's interactive built-in editor (scm-record) for splitting commits and resolving conflicts.

## Overview

Jujutsu includes a built-in TUI (Text User Interface) editor called **scm-record** for interactive change selection. It's used for:
- `jj split` - Splitting commits
- `jj squash -i` - Interactive squashing
- `jj resolve` - Resolving conflicts

## When You See It

The built-in editor appears in these situations:

```bash
# Splitting a commit
jj split
# → Editor opens with all changes

# Interactive squashing
jj squash -i
# → Editor opens to choose what to squash

# Resolving conflicts (if configured)
jj resolve
# → Editor opens with conflict sides
```

## Interface Overview

```
┌─────────────────────────────────────────────────┐
│ File List (left)    │ Diff View (right)         │
│                     │                            │
│ (+) src/main.rs ○   │ @@ -10,3 +10,7 @@         │
│ (+) src/lib.rs ○    │ -old line                 │
│ (+) README.md ○     │ +new line                 │
│                     │ +another new line          │
│                     │                            │
│                     │                            │
└─────────────────────────────────────────────────┘
  [q]uit [c]ommit [?]help

○ = unselected (going to 2nd commit)
● = selected (going to 1st commit)
```

## Basic Navigation

### Keyboard Shortcuts

| Key | Action |
|-----|--------|
| **Movement** ||
| `↑` / `k` | Move up |
| `↓` / `j` | Move down |
| `→` / `l` | Expand file / Move right |
| `←` / `h` | Collapse file / Move left |
| `g` | Go to top |
| `G` | Go to bottom |
| **Selection** ||
| `Space` | Toggle selection (file/hunk/line) |
| `a` | Select all in current file |
| `d` | Deselect all in current file |
| `A` | Select all files |
| `D` | Deselect all files |
| **Files** ||
| `Enter` | Expand/collapse file |
| `f` | Toggle file selection |
| **Navigation** ||
| `n` | Next file |
| `p` | Previous file |
| `]` | Next hunk |
| `[` | Previous hunk |
| **Actions** ||
| `c` | Commit selection (confirm) |
| `q` | Quit without saving |
| `?` | Show help |

### Mouse Support

- **Click** on files/hunks/lines to toggle selection
- **Scroll** to navigate
- **Click** menu items for actions

## Splitting Commits

### Basic Split Workflow

```bash
# Start split
jj split

# Interface opens:
# 1. All files are initially unselected (○)
# 2. Select what goes in FIRST commit
# 3. Rest goes in SECOND commit

# Example:
# Want to split:
#   file-a.txt → First commit
#   file-b.txt → Second commit

# Navigate to file-a.txt
j j  # Move down

# Select it
Space  # Now it shows ●

# Commit selection
c

# Result: Two commits created
```

### Selecting Hunks

```bash
jj split

# Navigate to a file
j j  # Move down
Enter  # or → to expand file

# You see hunks (sections of changes):
# ● Hunk 1: Lines 10-15
# ○ Hunk 2: Lines 20-25
# ○ Hunk 3: Lines 30-35

# Toggle individual hunks
Space  # Toggle current hunk
]  # Next hunk
Space  # Toggle this one

# Commit when ready
c
```

### Selecting Individual Lines

```bash
jj split

# Navigate to file and expand
j j Enter

# Expand hunk to see lines
→  # or l

# Individual lines shown:
# ● +new line 1
# ○ +new line 2
# ○ +new line 3

# Toggle each line
Space  # Toggle line
j  # Move down
Space  # Toggle next line

# Commit selection
c
```

## Interactive Squashing

### Basic Squash Workflow

```bash
# Interactively choose what to squash into parent
jj squash -i

# Similar interface to split
# Select what to move to parent
# Unselected stays in current commit

# Example:
# Current commit has:
#   feature.rs (want to squash)
#   test.rs (keep in current commit)

# Select feature.rs
Space

# Commit (move to parent)
c
```

## Conflict Resolution

### Using Built-in Editor for Conflicts

```bash
# Configure to use built-in editor
jj config set --user ui.merge-editor ":builtin"

# Resolve conflict
jj resolve

# Editor shows:
# - Left side (your changes)
# - Base (common ancestor)
# - Right side (their changes)

# Select which parts to keep
# Navigate and select with Space

# Commit resolution
c
```

## Advanced Usage

### Selecting All in File

```bash
# Navigate to file
j j  # Move to file

# Select all changes in this file
a  # Select all

# Or deselect all
d  # Deselect all
```

### Selecting All Files

```bash
# Select every file
A  # (Capital A)

# Deselect every file
D  # (Capital D)

# Then toggle individual files as needed
j j Space  # Deselect this one
j j Space  # Deselect another
```

### Quick File Navigation

```bash
# Jump between files
n  # Next file
p  # Previous file

# Go to extremes
g  # Top of list
G  # Bottom of list
```

### Viewing Changes

```bash
# Expand file to see changes
Enter  # or →

# Collapse file
Enter  # or ←

# Navigate through hunks
]  # Next hunk
[  # Previous hunk
```

## Practical Examples

### Example 1: Split Test from Implementation

```bash
# You have one commit with:
# - src/feature.rs (implementation)
# - tests/test_feature.rs (tests)

jj split

# Select implementation file
j  # Move to src/feature.rs
Space  # Select it (●)

# Commit
c

# Result:
# Commit 1: src/feature.rs (implementation)
# Commit 2: tests/test_feature.rs (tests)

# Describe each
jj describe -m "Implement feature X"
jj edit @-
jj describe -m "Add tests for feature X"
```

### Example 2: Split by Logical Changes

```bash
# One file with two unrelated changes
jj split

# Navigate to the file
j  # Move down
Enter  # Expand file

# See hunks:
# Hunk 1: Add function foo (lines 10-20)
# Hunk 2: Fix bug in bar (lines 50-55)

# Select only first hunk
Space  # Select hunk 1
]  # Move to next hunk
# (Don't select hunk 2)

# Commit first change
c

# Result:
# Commit 1: Add function foo
# Commit 2: Fix bug in bar
```

### Example 3: Cherry-Pick Lines

```bash
# Want to keep only specific lines in first commit
jj split

j j Enter  # Navigate to file and expand
→  # Expand hunk to show lines

# Individual lines:
# Line 1: +import new_module
# Line 2: +def helper():
# Line 3: +    pass
# Line 4: +def main_function():
# Line 5: +    use_helper()

# Select only the import
Space  # Select line 1
j j j j  # Skip other lines

# Commit
c

# Result:
# Commit 1: Just the import
# Commit 2: Function definitions
```

### Example 4: Interactive Squash Specific Files

```bash
# Current commit has many files
# Want to squash only bug fixes to parent

jj squash -i

# Navigate and select only bug fix files
j  # Move to bugfix-1.rs
Space  # Select
j j j  # Move to bugfix-2.rs
Space  # Select
# Skip feature files

# Commit (squash selected to parent)
c

# Result:
# Parent: Now has bug fixes
# Current: Still has feature work
```

## Tips and Tricks

### Tip 1: Preview Before Confirming

Before pressing `c`, review your selections:
- Files with `●` go to first commit / squash destination
- Files with `○` stay in second commit / current

### Tip 2: Undo If Mistake

```bash
# If you made mistake after confirming
jj undo  # Undo the split/squash

# Try again
jj split  # Or jj squash -i
```

### Tip 3: Use Expand/Collapse

```bash
# See file list easily
h h h  # Collapse all files (if expanded)

# Focus on one file
j j  # Navigate to it
l  # Expand just this one

# Expand all files (rarely useful)
# Navigate to each and press Enter
```

### Tip 4: Quick Select All Then Deselect

```bash
# Want to select most files except a few
A  # Select all
j j  # Navigate to exception
Space  # Deselect it
j j j  # Navigate to another
Space  # Deselect it

# Commit the selection
c
```

### Tip 5: Mouse for Quick Selection

```bash
# Use mouse to click through files quickly
# Click on:
# - File names to toggle file
# - Hunks to toggle hunk
# - Lines to toggle line

# Then 'c' to commit
```

## Configuration

### Default Editor Setting

```toml
# ~/.config/jj/config.toml
[ui]
diff-editor = ":builtin"  # Use built-in (default)

# Or use external:
# diff-editor = "code --wait"
# diff-editor = "vim -d"
```

### Merge Editor

```toml
[ui]
merge-editor = ":builtin"  # Use built-in for conflicts

# Or external:
# merge-editor = "meld"
```

## Troubleshooting

### Problem: Can't See Changes

```bash
# Make sure file is expanded
Enter  # Expand file

# Make sure hunk is expanded
→  # Expand hunk to see lines
```

### Problem: Selection Not Working

```bash
# Ensure you're on the right level:
# - File level: Select entire file
# - Hunk level: Select hunk
# - Line level: Select individual line

# Navigate to right level with ← →
```

### Problem: Editor Doesn't Open

```bash
# Check configuration
jj config get ui.diff-editor

# Set to builtin
jj config set --user ui.diff-editor ":builtin"
```

### Problem: Terminal Display Issues

```bash
# Try different terminal emulator
# Or check terminal supports colors:
echo $TERM

# Force color mode
jj split --config-toml='ui.color="always"'
```

## Alternative Editors

If built-in editor doesn't suit you:

```toml
# ~/.config/jj/config.toml

# VS Code
[ui]
diff-editor = "code --wait --diff"

# Vim
[ui]
diff-editor = "vim -d"

# Emacs
[ui]
diff-editor = "emacs --diff"

# Difftastic (view only, not interactive)
[ui.diff]
tool = "difftastic"
```

## Summary

| Task | Command | In Editor |
|------|---------|-----------|
| **Split commit** | `jj split` | Select → `c` |
| **Interactive squash** | `jj squash -i` | Select → `c` |
| **Resolve conflict** | `jj resolve` | Select → `c` |
| **Toggle selection** | | `Space` |
| **Expand file** | | `Enter` or `→` |
| **Navigate** | | `↑↓` or `jk` |
| **Commit selection** | | `c` |
| **Quit** | | `q` |

## Best Practices

### DO

✓ Expand files to see exactly what you're selecting
✓ Use hunk-level selection for coarse splitting
✓ Use line-level selection for precise splitting
✓ Press `?` to see help when stuck
✓ Use `jj undo` if you make a mistake

### DON'T

✗ Rush through without reviewing selections
✗ Select without expanding to see changes
✗ Forget that `●` means "included in first commit"
✗ Panic - you can always `jj undo`

## Next Steps

- [Advanced Features](Advanced-Features.md) - More split/squash techniques
- [Workflows](Workflows.md) - When to split commits
- [Best Practices](Best-Practices.md) - Modern commit organization

---

[← Back to Advanced Features](Advanced-Features.md) | [Main](README.md) | [Next: Best Practices →](Best-Practices.md)
