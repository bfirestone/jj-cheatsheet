# Getting Started with Jujutsu

Your first steps with Jujutsu VCS - from initialization to your first change.

## Prerequisites

- [Jujutsu installed](Installation.md)
- Basic familiarity with version control concepts
- Optional: Git installed (for Git interoperability)

## Your First Repository

### Option 1: Start with an Existing Git Repository (Recommended)

The easiest way to try jj is with an existing Git repository using **colocated mode**:

```bash
# Navigate to your Git repository
cd /path/to/your/git/repo

# Initialize jj in colocated mode
jj git init --colocate

# You can now use both git and jj commands!
```

This creates a `.jj` directory alongside your `.git` directory. Both tools work on the same repository.

### Option 2: Clone a Git Repository

```bash
# Clone a repository with jj
jj git clone --colocate https://github.com/user/repo.git

# Or use SSH
jj git clone --colocate git@github.com:user/repo.git
```

### Option 3: Create a New Repository

```bash
# Create a new directory
mkdir my-project
cd my-project

# Initialize as a jj repository with Git backend
jj git init

# Or use colocated mode (recommended)
jj git init --colocate
```

### Option 4: Pure jj Repository (No Git)

```bash
mkdir my-project
cd my-project
jj init
```

**Note**: Most users should use Git-backed repositories for compatibility with remotes.

## Understanding Your Repository State

After initialization, check your repository status:

```bash
# View the current state
jj status

# Example output:
# Working copy : qpvuntsm 4e411c1a (empty) (no description set)
# Parent commit: zzzzzzzz 00000000 (empty) (no description set)
```

### What This Means

- **Working copy**: Your current working state (always a commit in jj!)
- **Parent commit**: The commit you're building on
- **Change ID** (e.g., `qpvuntsm`): A unique identifier that persists across rebases
- **Commit ID** (e.g., `4e411c1a`): The actual commit hash (changes with content)

## Your First Change

### Traditional Workflow (Like Git)

```bash
# 1. Create a new change
jj new -m "Add hello world example"

# 2. Edit files
echo "print('Hello, world!')" > hello.py

# 3. Check status
jj status
# Output shows hello.py as added

# 4. View what changed
jj diff

# 5. Squash changes into the current change
jj squash
```

### Modern Workflow (Recommended)

In jj, you can describe what you're going to do first:

```bash
# 1. Create a new change with description
jj new -m "Add hello world example"

# 2. Make your changes
echo "print('Hello, world!')" > hello.py

# 3. The changes are automatically in the current change!
jj status  # Shows changes in current commit

# 4. Continue to next change
jj new -m "Add README"
echo "# My Project" > README.md
```

**Key Insight**: Your working directory is always a commit. No need to "stage" changes!

## Viewing History

```bash
# View commit history
jj log

# Example output:
# @  qpvuntsm ben@example.com 2025-10-15 10:23:45 4e411c1a
# │  Add hello world example
# ◉  zzzzzzzz root() 00000000
#    (empty) (no description set)
```

### Understanding the Log

- `@` - Your current working copy
- `◉` - Other commits
- `│` - Ancestry lines
- Change IDs and commit IDs
- Author, timestamp, and description

### Customizing Log Output

```bash
# Show more commits
jj log -r 'all()'

# Show specific range
jj log -r 'main@origin..'

# Compact format
jj log --no-graph

# More details
jj show
```

## Making More Changes

### Linear History

```bash
# Create a change
jj new -m "Add feature A"
# Edit files...

# Create another change on top
jj new -m "Add feature B"
# Edit files...

# View the stack
jj log
```

### Parallel Changes (Branching)

```bash
# Go back to a previous change
jj edit @-  # Or use a change ID

# Create a new change from here
jj new -m "Alternative approach"
# Edit files...

# You now have parallel changes!
jj log  # Shows the branch
```

## Working with Remotes

### Fetching Changes

```bash
# Fetch from the default remote
jj git fetch

# Fetch from a specific remote
jj git fetch --remote origin

# Fetch all remotes
jj git fetch --all-remotes
```

### Pushing Changes

```bash
# Create a bookmark (like a Git branch)
jj bookmark create my-feature

# Push the bookmark
jj git push --bookmark my-feature

# Or push all bookmarks
jj git push --all
```

### Tracking Remote Changes

```bash
# See remote bookmarks
jj bookmark list --all

# Track a remote bookmark
jj bookmark track main@origin

# Rebase onto latest main
jj rebase -d main@origin
```

## Essential Commands Summary

| Command | Purpose |
|---------|---------|
| `jj status` | Show working copy status |
| `jj log` | View commit history |
| `jj new -m "msg"` | Create new change with description |
| `jj edit <rev>` | Switch to a different change |
| `jj diff` | Show changes in working copy |
| `jj show` | Show details of current change |
| `jj squash` | Move changes into parent |
| `jj describe` | Edit change description |
| `jj undo` | Undo last operation |

## Common First-Time Questions

### Q: Where's the staging area?

**A**: There isn't one! Your working directory is always a commit. Changes are automatically tracked.

### Q: How do I commit?

**A**: You don't "commit" in the Git sense. Instead:
1. Create a new change with `jj new -m "description"`
2. Make your edits
3. The changes are already in that change!

### Q: How do I amend?

**A**: Just edit the change you want to modify:
```bash
jj edit <change-id>  # Switch to the change
# Make edits...
# They're automatically in that change!
```

### Q: What if I make a mistake?

**A**: Use `jj undo`! It undoes the last operation. You can undo multiple times.

```bash
jj undo  # Undo last operation
jj undo  # Undo the one before that
```

### Q: Can I still use Git commands?

**A**: Yes! In colocated mode, you can use both:
```bash
git status  # Works!
jj status   # Also works!
```

However, prefer `jj` commands for making changes to avoid confusion.

### Q: How do I create a branch?

**A**: In jj, they're called "bookmarks":
```bash
jj bookmark create feature-name
jj bookmark list  # See all bookmarks
```

## Example Workflow: Making a Pull Request

```bash
# 1. Start fresh from main
jj git fetch
jj new main@origin -m "Add new feature"

# 2. Make changes
echo "new feature" > feature.txt

# 3. Create a bookmark for PR
jj bookmark create new-feature

# 4. Push to remote
jj git push --bookmark new-feature

# 5. Create PR using GitHub CLI or web interface
gh pr create --title "Add new feature" --body "Description"
```

## Example Workflow: Fixing a Bug

```bash
# 1. Create a change for the fix
jj new -m "Fix: Resolve null pointer issue"

# 2. Make the fix
# Edit files...

# 3. Test your fix
./run-tests.sh

# 4. If tests fail, edit more
# Files automatically update the same change!

# 5. Describe the fix better if needed
jj describe  # Opens editor

# 6. Push the fix
jj bookmark create bugfix-123
jj git push --bookmark bugfix-123
```

## Example Workflow: Exploring History

```bash
# View recent history
jj log

# Look at a specific change
jj show <change-id>

# See what changed in a file
jj diff -r <change-id> path/to/file

# Go back in time
jj edit <change-id>
# Look around, test things...

# Return to latest
jj edit @-  # Or use the change ID
```

## Tips for Git Users

### Mental Model Shift

| Git Concept | Jujutsu Concept |
|-------------|-----------------|
| Staging area | None - working copy is a commit |
| Branch | Bookmark (less important) |
| HEAD | `@` (working copy) |
| Commit | Change (mutable until pushed) |
| Rebase | Automatic in many cases |
| Stash | Not needed - just edit any change |

### Common Translations

```bash
# Git → Jujutsu

# git add -p; git commit -m "msg"
jj new -m "msg"  # Then edit files

# git commit --amend
jj describe  # Or just edit files in the current change

# git rebase -i HEAD~3
jj log  # Then jj squash, jj rebase, etc.

# git stash
jj edit <other-change>  # Switch to any change

# git cherry-pick <commit>
jj duplicate <change>  # Or jj rebase

# git reset --hard HEAD~1
jj abandon @  # Or jj undo
```

## Next Steps

Now that you understand the basics:

1. [Core Concepts](Core-Concepts.md) - Deepen your understanding
2. [Workflows](Workflows.md) - Learn common patterns
3. [Commands Reference](Commands-Reference.md) - Explore all commands
4. [Git Comparison](Git-Comparison.md) - Complete Git command mapping

## Practice Exercise

Try this sequence to get comfortable:

```bash
# 1. Initialize a test repository
mkdir jj-practice
cd jj-practice
jj git init --colocate

# 2. Create a series of changes
jj new -m "First change"
echo "first" > file.txt

jj new -m "Second change"
echo "second" >> file.txt

jj new -m "Third change"
echo "third" >> file.txt

# 3. View your stack
jj log

# 4. Go back and modify the first change
jj edit @--  # Two parents back
echo "modified first" > file.txt

# 5. See how descendants were automatically rebased!
jj log

# 6. Experiment with undo
jj undo  # Undo the edit
jj log   # Back to before

# 7. Clean up
cd ..
rm -rf jj-practice
```

---

[← Back to Installation](Installation.md) | [Main](README.md) | [Next: Core Concepts →](Core-Concepts.md)
