# Git vs Jujutsu Command Comparison

Quick reference for Git users learning Jujutsu - command equivalents and conceptual mappings.

## Philosophy Differences

| Aspect | Git | Jujutsu |
|--------|-----|---------|
| **Working Directory** | Separate from commits | Is a commit |
| **Staging Area** | Required for commits | None - automatic |
| **Branches** | Central concept | Optional (bookmarks) |
| **Commit Mutability** | Avoid changing history | Encouraged until pushed |
| **Conflicts** | Block operations | First-class, don't block |
| **Undo** | Complex (reflog, reset) | Simple (`jj undo`) |

## Command Translation Table

### Repository Setup

| Git | Jujutsu | Notes |
|-----|---------|-------|
| `git init` | `jj git init --colocate` | Creates Git-backed repo |
| `git clone <url>` | `jj git clone --colocate <url>` | Colocated mode recommended |
| `git status` | `jj status` | Similar output |
| `git log` | `jj log` | More visual by default |
| `git remote add <name> <url>` | `jj git remote add <name> <url>` | Same syntax |

### Viewing Changes

| Git | Jujutsu | Notes |
|-----|---------|-------|
| `git log` | `jj log` | Default shows graph |
| `git log --oneline` | `jj log --no-graph` | Compact view |
| `git log --graph` | `jj log` | Graph is default |
| `git show <commit>` | `jj show <commit>` | Same concept |
| `git diff` | `jj diff` | Shows working copy changes |
| `git diff HEAD` | `jj diff` | jj has no staging |
| `git diff --staged` | N/A | No staging area in jj |
| `git diff <commit>` | `jj diff -r <commit>` | Specific commit diff |
| `git diff A..B` | `jj diff --from A --to B` | Range diff |
| `git log --follow <file>` | `jj log <file>` | File history |
| `git blame <file>` | `jj file annotate <file>` | Line-by-line history |

### Making Changes

| Git | Jujutsu | Notes |
|-----|---------|-------|
| `git add <file>` | N/A | Changes are auto-tracked |
| `git add -p` | `jj split` or `jj squash -i` | Interactive selection |
| `git commit -m "msg"` | `jj new -m "msg"` → edit → `jj squash` | Or just edit and `jj describe` |
| `git commit -a -m "msg"` | `jj describe -m "msg"` && `jj new` | Working copy is always committed |
| `git commit --amend` | Just edit files! | Working copy is mutable |
| `git commit --amend --no-edit` | Just edit files! | No separate command needed |
| `git add . && git commit` | `jj describe -m "msg"` | Working copy is the commit |
| `git reset HEAD <file>` | N/A | No staging to reset |
| `git reset --soft HEAD~1` | `jj abandon @` | Remove last commit |
| `git reset --hard HEAD~1` | `jj abandon @` or `jj edit @-` | Choose based on need |
| `git reset --hard <commit>` | `jj edit <commit>` | Move working copy |
| `git clean -fd` | `jj abandon @` then `jj new` | Remove untracked |

### Branching & Bookmarks

| Git | Jujutsu | Notes |
|-----|---------|-------|
| `git branch` | `jj bookmark list` | List branches/bookmarks |
| `git branch <name>` | `jj bookmark create <name>` | Create branch/bookmark |
| `git switch <branch>` | `jj edit <bookmark>` | Switch branches |
| `git checkout <branch>` | `jj edit <bookmark>` | Switch branches |
| `git checkout -b <name>` | `jj bookmark create <name>` | Create and switch |
| `git branch -d <name>` | `jj bookmark delete <name>` | Delete branch |
| `git branch -m <old> <new>` | `jj bookmark rename <old> <new>` | Rename |
| `git branch --set-upstream-to=<remote>/<branch>` | `jj bookmark track <branch>@<remote>` | Track remote |
| `git checkout <commit>` | `jj edit <commit>` | Detached HEAD → just editing commit |

### Remote Operations

| Git | Jujutsu | Notes |
|-----|---------|-------|
| `git fetch` | `jj git fetch` | Fetch from remote |
| `git fetch --all` | `jj git fetch --all-remotes` | All remotes |
| `git pull` | `jj git fetch && jj rebase -d <branch>@<remote>` | No pull command |
| `git pull --rebase` | `jj git fetch && jj rebase -d <branch>@<remote>` | Default is rebase-like |
| `git push` | `jj git push --bookmark <name>` | Must specify bookmark |
| `git push -u origin <branch>` | `jj git push --bookmark <name>` | Creates tracking |
| `git push --force` | `jj git push --bookmark <name> --force` | Force push |
| `git push --all` | `jj git push --all` | Push all bookmarks |

### History Rewriting

| Git | Jujutsu | Notes |
|-----|---------|-------|
| `git rebase <base>` | `jj rebase -d <base>` | Rebase current change |
| `git rebase -i HEAD~3` | `jj log` then various commands | More granular tools |
| `git rebase --continue` | N/A | Conflicts don't block |
| `git rebase --abort` | `jj undo` | Undo operation |
| `git cherry-pick <commit>` | `jj duplicate <commit>` or `jj rebase` | Create copy or move |
| `git revert <commit>` | `jj backout <commit>` | Create inverse commit |
| `git commit --fixup <commit>` | `jj squash --into <commit>` | Fix up commit |
| `git rebase --autosquash` | `jj absorb` | Automatic fixup |

### Squashing & Splitting

| Git | Jujutsu | Notes |
|-----|---------|-------|
| `git rebase -i` (squash) | `jj squash` | Squash into parent |
| `git reset HEAD~1` then `git add -p` | `jj split` | Split commit |
| `git commit --amend` | `jj squash` or just edit | Amend is just squashing |

### Merging

| Git | Jujutsu | Notes |
|-----|---------|-------|
| `git merge <branch>` | `jj merge <branch1> <branch2>` | Explicit two parents |
| `git merge --abort` | `jj undo` | Undo operation |
| `git merge --continue` | N/A | Conflicts don't block |
| `git mergetool` | `jj resolve` | Resolve conflicts |

### Stashing

| Git | Jujutsu | Notes |
|-----|---------|-------|
| `git stash` | `jj new` or `jj edit <other>` | Just work on different change |
| `git stash pop` | `jj edit <change>` | Switch back to change |
| `git stash list` | `jj log` | See all changes |
| `git stash apply` | `jj duplicate <change>` | Copy change |
| `git stash drop` | `jj abandon <change>` | Remove change |

### Undoing & Recovery

| Git | Jujutsu | Notes |
|-----|---------|-------|
| `git reflog` | `jj op log` | Operation history |
| `git reset --hard <commit>` | `jj op restore <operation>` | Restore to state |
| `git reflog` → `git reset` | `jj undo` | Much simpler! |
| `git fsck --lost-found` | `jj op log` | Find lost changes |

### Tags

| Git | Jujutsu | Notes |
|-----|---------|-------|
| `git tag <name>` | Not yet fully supported | Use Git for tags in colocated repos |
| `git tag -a <name>` | Not yet fully supported | Use Git |
| `git push --tags` | Use git in colocated repo | Use Git |

### Configuration

| Git | Jujutsu | Notes |
|-----|---------|-------|
| `git config --global user.name "Name"` | `jj config set --user user.name "Name"` | Similar |
| `git config --global user.email "email"` | `jj config set --user user.email "email"` | Similar |
| `git config --list` | `jj config list` | Show all config |

### Inspection

| Git | Jujutsu | Notes |
|-----|---------|-------|
| `git show <commit>:<file>` | `jj file show -r <commit> <file>` | Show file at revision |
| `git ls-files` | `jj file list` | List tracked files |
| `git rev-parse HEAD` | `jj log -r @ --no-graph` | Get commit ID |

## Workflow Translations

### Git Workflow: Feature Branch

```bash
# Git
git checkout -b feature
# edit files
git add .
git commit -m "Add feature"
git push -u origin feature

# Jujutsu
jj new main -m "Add feature"
# edit files (automatically in commit!)
jj bookmark create feature
jj git push --bookmark feature
```

### Git Workflow: Fixing Last Commit

```bash
# Git
git add <forgotten-file>
git commit --amend --no-edit

# Jujutsu
# edit files (they're automatically in the current commit!)
# That's it! No command needed.
```

### Git Workflow: Interactive Rebase

```bash
# Git
git rebase -i HEAD~3
# Interactive editor: pick, squash, edit, etc.

# Jujutsu (more granular)
jj log                    # See what you have
jj squash -r <change>     # Squash specific change
jj edit <change>          # Edit specific change
jj rebase -r <change> -d <dest>  # Rebase specific change
# Each operation is separate and undoable
```

### Git Workflow: Stash and Switch

```bash
# Git
git stash
git checkout other-branch
# do work
git checkout original-branch
git stash pop

# Jujutsu
jj edit <other-change>
# do work
jj edit <original-change>
# That's it! Changes are preserved automatically
```

### Git Workflow: Cherry-Pick

```bash
# Git
git checkout target-branch
git cherry-pick abc123

# Jujutsu
jj duplicate abc123
jj rebase -d target-bookmark
```

### Git Workflow: Undo Last Commit

```bash
# Git
git reset --hard HEAD~1
# or
git reset --soft HEAD~1

# Jujutsu
jj abandon @
# or
jj edit @-
```

## Common Git Patterns in Jujutsu

### Commit Some Files, Not Others

```bash
# Git
git add file1.txt file2.txt
git commit -m "Partial commit"

# Jujutsu
jj split file1.txt file2.txt
jj describe -m "Partial commit"
```

### Fix Up Multiple Commits

```bash
# Git
git commit --fixup=abc123
git commit --fixup=def456
git rebase -i --autosquash main

# Jujutsu
# Just make your edits in working copy
jj absorb  # Automatically distributes changes!
```

### Work on Multiple Features Simultaneously

```bash
# Git
git stash
git checkout -b feature2
# work
git stash
git checkout feature1
# work
git checkout feature2
git stash pop

# Jujutsu
jj new feature1 -m "Feature 1"
# work
jj new feature2 -m "Feature 2"  # Parallel branch
# work
jj edit feature1  # Switch anytime
jj edit feature2  # Switch anytime
```

## Conceptual Differences

### The Index/Staging Area

```
Git:
Working Dir → [git add] → Staging → [git commit] → Commit

Jujutsu:
Working Copy Commit (always committed)
```

**In jj**: There's no staging. Your working directory IS a commit. When you edit files, you're editing that commit directly.

### Branches vs Bookmarks

```
Git:
- Branches are essential
- Must create branch before committing
- Branch moves with each commit

Jujutsu:
- Bookmarks are optional
- Create changes first, bookmark later
- Bookmarks are just pointers, don't move automatically
```

### Detached HEAD

```
Git:
git checkout abc123  → Detached HEAD (scary!)
Make commits → Need to create branch or lose them

Jujutsu:
jj edit abc123  → Just editing that commit (normal!)
Make changes → They're tracked with change IDs
```

**In jj**: There's no "detached HEAD" state. Every change has a change ID and is tracked.

### Rebasing

```
Git:
- Must resolve conflicts before continuing
- Rebase can be aborted mid-way
- Easy to get into confusing states

Jujutsu:
- Automatic for descendants when you edit a commit
- Conflicts are recorded, don't block
- Always in a consistent state
```

## Things That Don't Have Git Equivalents

### Change IDs
```bash
jj log
# Shows both change IDs (persistent) and commit IDs (content hash)
```

Git only has commit IDs. Jujutsu's change IDs survive rebases and amendments.

### Operation Log
```bash
jj op log      # Complete history of all operations
jj op restore  # Restore to any previous state
```

Git's reflog is per-ref. Jujutsu's operation log is global and comprehensive.

### First-Class Conflicts
```bash
# In jj, you can commit conflicts!
jj rebase -d main   # Creates conflict
jj log              # Shows (conflict) marker
jj new              # Continue working
jj git push         # Can even push if you want
# Resolve later when ready
```

Git forces you to resolve before continuing.

### Working Copy as Commit
```bash
# In jj, @ is always a commit
jj show @   # Show current working copy as commit
```

Git's working directory is separate from commits.

## Migration Checklist

- [ ] Install jujutsu
- [ ] Set up config (name, email)
- [ ] Initialize colocated repo or clone
- [ ] Understand working copy as commit
- [ ] Practice `jj new`, `jj edit`, `jj squash`
- [ ] Learn `jj undo` for safety
- [ ] Use `jj log` frequently
- [ ] Create bookmarks only when pushing
- [ ] Embrace mutable history
- [ ] Explore `jj absorb` and `jj split`

## Tips for Git Users

1. **Stop thinking about staging** - Your working directory is the commit
2. **Don't create branches early** - Create changes first, bookmark later
3. **Edit history freely** - It's encouraged, not dangerous
4. **Use `jj undo` liberally** - It's your safety net
5. **Let go of "detached HEAD" fear** - Doesn't exist in jj
6. **Embrace `jj log`** - Use it constantly to see where you are
7. **Try `jj absorb`** - It's magic for fixing multiple commits
8. **Don't fear conflicts** - They don't block you

## Common Mistakes

### Mistake 1: Creating bookmarks too early
```bash
# Don't do this (Git habit):
jj bookmark create feature
jj new -m "Add feature"

# Do this instead:
jj new -m "Add feature"
# ... work on feature ...
jj bookmark create feature  # Only when ready to push
```

### Mistake 2: Looking for staging
```bash
# Don't try to do this:
# jj add file.txt  ← Doesn't exist!

# Instead:
jj split file.txt  # If you want to separate changes
# or
jj squash -i       # Interactive selection
```

### Mistake 3: Fear of editing history
```bash
# Don't be afraid to:
jj edit @---   # Go back in history
# Make changes
jj edit @      # Come back
# Descendants are automatically rebased!
```

### Mistake 4: Not using `jj undo`
```bash
# Did something wrong?
jj undo  # Just undo it!
# Try again
```

## Next Steps

- [Git Interoperability](Git-Interop.md) - Using jj and Git together
- [Workflows](Workflows.md) - Modern jj workflows
- [Commands Reference](Commands-Reference.md) - Detailed command docs

---

[← Back to Commands Reference](Commands-Reference.md) | [Main](README.md) | [Next: Git Interop →](Git-Interop.md)
