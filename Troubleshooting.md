# Troubleshooting

Solutions to common issues and problems when using Jujutsu.

## Installation Issues

### Problem: `jj: command not found`

After installing jj, the command isn't recognized.

**Solution:**

```bash
# Check if jj is installed
which jj

# If not found, check installation method:

# For cargo install:
# Add cargo bin to PATH
export PATH="$HOME/.cargo/bin:$PATH"
# Add to ~/.bashrc or ~/.zshrc for persistence

# For package managers:
# Reinstall
brew install jj  # macOS
sudo pacman -S jujutsu  # Arch
```

### Problem: Build from Source Fails

Cargo build fails with errors.

**Solution:**

```bash
# Update Rust
rustup update stable

# Install dependencies
# Ubuntu/Debian:
sudo apt install build-essential pkg-config libssl-dev

# macOS:
brew install pkg-config openssl

# Clean and rebuild
cargo clean
cargo build --release
```

### Problem: Version Mismatch

jj version is outdated.

**Solution:**

```bash
# Check current version
jj --version

# Update:
# Via cargo:
cargo install --locked jj-cli --force

# Via package manager:
brew upgrade jj  # macOS
sudo pacman -Syu jujutsu  # Arch
```

## Configuration Issues

### Problem: Config File Not Loading

Changes to config file don't take effect.

**Solution:**

```bash
# Check config file location
jj config path

# Verify config file exists
ls -la ~/.config/jj/config.toml  # Linux/macOS

# Check for syntax errors
jj config list
# If error, fix TOML syntax

# Verify specific setting
jj config get user.name
```

### Problem: TOML Syntax Errors

Config file has invalid TOML.

**Solution:**

```toml
# Common mistakes:

# ❌ Wrong: Missing closing bracket
[user
name = "Test"

# ✓ Correct:
[user]
name = "Test"

# ❌ Wrong: Quotes in section name
["user-settings"]

# ✓ Correct:
[user-settings]

# ❌ Wrong: Missing quotes around string
[user]
name = Test

# ✓ Correct:
[user]
name = "Test"
```

### Problem: Editor Not Opening

jj describe doesn't open your editor.

**Solution:**

```bash
# Set editor explicitly
jj config set --user ui.editor "vim"

# Or use environment variable
export EDITOR=vim
export VISUAL=vim

# For VS Code:
jj config set --user ui.editor "code --wait"

# Test it
jj describe  # Should open editor
```

## Repository Issues

### Problem: "Not a jj repository"

Running jj commands outside a repository.

**Solution:**

```bash
# Check if in repository
ls -la .jj  # Should exist

# If not, initialize:
jj git init --colocate

# Or navigate to repository:
cd /path/to/your/repo
```

### Problem: Colocated Repository Conflicts

Git and jj show different states.

**Solution:**

```bash
# Force sync
jj status  # Triggers import from Git

# Or explicitly export/import
jj git export  # Export jj state to Git
jj git import  # Import Git state to jj

# Check for conflicts
jj bookmark list
# Look for "conflict" status

# Resolve bookmark conflicts
jj bookmark set <bookmark> -r <correct-revision>
```

### Problem: Large Repository Performance

Operations are slow in large repository.

**Solution:**

```bash
# Option 1: Use non-colocated mode
jj git clone <url>  # Without --colocate

# Option 2: Clean up old bookmarks
git branch -d old-feature
git remote prune origin

# Option 3: Clean up operation log
# Wait for auto-GC, or:
jj config set --user operation.gc-after-days 30

# Option 4: Use shallow clone
git clone --depth 1 <url>
cd repo
jj git init --colocate
```

## Working Copy Issues

### Problem: Working Copy Diverged

Working copy out of sync with repository.

**Solution:**

```bash
# Snapshot current state
jj status  # Forces snapshot

# If that doesn't work:
jj op log  # Find last good operation
jj op restore <operation-id>

# Or abandon and restart
jj new
```

### Problem: Can't Edit Files

Files appear read-only or changes don't register.

**Solution:**

```bash
# Check file permissions
ls -la file.txt

# Ensure not in read-only mode
# Check if disk is full
df -h .

# Force snapshot
jj debug snapshot

# If all else fails
jj undo
jj new
```

### Problem: Untracked Files Not Showing

New files aren't showing in `jj status`.

**Solution:**

```bash
# Check .gitignore
cat .gitignore
# Ensure file isn't ignored

# Explicitly track
jj file track path/to/file

# Force snapshot
jj status --refresh
```

## Sync and Remote Issues

### Problem: Push Rejected

Remote rejects your push.

**Solution:**

```bash
# Fetch latest
jj git fetch

# Rebase your work
jj rebase -d main@origin

# If bookmark moved forward on remote:
jj bookmark set feature -r <your-revision>
jj git push --bookmark feature --force

# For force push (be careful!)
# Only do this on YOUR branches
jj git push --bookmark feature --force
```

### Problem: Fetch Fails

Can't fetch from remote.

**Solution:**

```bash
# Check remote configuration
jj git remote list

# Test with git
git fetch  # In colocated repo

# Check network/credentials
git ls-remote origin

# Re-add remote if needed
jj git remote remove origin
jj git remote add origin <url>

# Fetch again
jj git fetch --remote origin
```

### Problem: Bookmark Not Pushing

Bookmark doesn't appear on remote.

**Solution:**

```bash
# Ensure bookmark exists locally
jj bookmark list

# Create if needed
jj bookmark create feature

# Push explicitly
jj git push --bookmark feature

# Check remote after push
jj git fetch
jj bookmark list --all
```

## Conflict Resolution Issues

### Problem: Can't Resolve Conflicts

Conflicts seem impossible to resolve.

**Solution:**

```bash
# Option 1: Undo and try different approach
jj undo
jj rebase -r @ -d main@origin  # Rebase one commit at a time

# Option 2: Take one side entirely
jj resolve --tool :local file.txt  # Take ours
# or
jj resolve --tool :other file.txt  # Take theirs

# Option 3: Start over
jj undo  # Undo the rebase
# Try merging instead
jj merge @ main@origin
```

### Problem: Conflict Markers Confusing

Can't understand jj's conflict format.

**Solution:**

```bash
# Use external tool
jj config set --user ui.merge-editor "meld"
jj resolve --tool meld file.txt

# Or use simpler markers
jj config set --user ui.conflict-marker-style "diff3"

# View each side separately
jj file show -r 'parents(@)[0]' file.txt  # Left side
jj file show -r 'parents(@)[1]' file.txt  # Right side
```

### Problem: Conflicts Keep Reappearing

Resolved conflicts come back.

**Solution:**

```bash
# Ensure you're resolving in correct commit
jj log  # Find conflicted commit
jj edit <conflict-commit-id>
jj resolve

# After resolving, verify
jj status  # Should show no conflicts

# If conflicts are in descendants:
# Resolve bottom-up
jj edit <earliest-conflict>
jj resolve
jj edit <next-conflict>
jj resolve
```

## History Issues

### Problem: Lost Commits

Can't find a commit you created.

**Solution:**

```bash
# Check operation log
jj op log -n 50

# Look for your commit in log
jj log -r 'all()'

# Find by description
jj log -r 'description(pattern:"your text")'

# Restore from operation log
jj op log  # Find when commit existed
jj op restore <operation-id>

# Or undo recent operations
jj undo
jj undo
# Keep undoing until you find it
```

### Problem: Accidentally Abandoned Commit

Ran `jj abandon` by mistake.

**Solution:**

```bash
# Immediately undo
jj undo

# If you didn't notice right away:
jj op log  # Find the abandon operation
jj op restore <operation-before-abandon>

# Or undo to that point
jj undo  # Keep running until commit returns
```

### Problem: Complex Rebase Broke Everything

After a complex rebase, history is a mess.

**Solution:**

```bash
# Find operation before rebase
jj op log
# Look for your rebase operation

# Restore to before it
jj op restore <operation-before-rebase>

# Try simpler approach:
# Rebase one commit at a time
jj rebase -r <commit1> -d main@origin
jj rebase -r <commit2> -d <commit1>
# etc.
```

## Operation Log Issues

### Problem: Operation Log Too Large

Operation log takes up too much space.

**Solution:**

```bash
# Check current setting
jj config get operation.gc-after-days

# Reduce retention period
jj config set --user operation.gc-after-days 30

# Manual cleanup (rarely needed)
# Let auto-GC handle it
```

### Problem: Can't Undo Anymore

Too many operations, can't find the right one.

**Solution:**

```bash
# View more operations
jj op log -n 100

# Filter operations
jj op log | grep "rebase"

# If too far back:
# Can't undo that far
# Manually recreate desired state
```

## Bookmark Issues

### Problem: Bookmark Conflict

Bookmark shows as conflicted.

**Solution:**

```bash
# View conflict
jj bookmark list
# feature: conflict
#   ??? abc123
#   ??? def456

# Choose correct revision
jj bookmark set feature -r abc123

# Or delete and recreate
jj bookmark delete feature
jj bookmark create feature -r abc123
```

### Problem: Can't Delete Bookmark

Bookmark deletion fails.

**Solution:**

```bash
# If tracked remotely, untrack first
jj bookmark untrack feature@origin

# Then delete local
jj bookmark delete feature

# Or forget (delete + untrack)
jj bookmark forget feature

# To delete from remote:
jj git push --bookmark feature --delete
```

### Problem: Bookmark Not Tracking Remote

Local bookmark not following remote changes.

**Solution:**

```bash
# Track the remote bookmark
jj bookmark track feature@origin

# Fetch to update
jj git fetch

# Check tracking status
jj bookmark list --tracked
```

## Diff and Merge Tool Issues

### Problem: Difftool Not Working

External diff tool doesn't open.

**Solution:**

```bash
# Check configuration
jj config get ui.diff.tool

# Verify tool is installed
which difft  # or meld, or delta

# Test with simple diff
jj diff

# Set tool explicitly
jj config set --user ui.diff.tool "difftastic"

# Use different tool
jj diff --tool vimdiff
```

### Problem: Merge Tool Not Opening

jj resolve doesn't open merge tool.

**Solution:**

```bash
# Check configuration
jj config get ui.merge-editor

# Set merge tool
jj config set --user ui.merge-editor "meld"

# Configure tool properly
jj config set --user merge-tools.meld.program "meld"

# Use specific tool
jj resolve --tool meld file.txt
```

## Performance Issues

### Problem: Commands Are Slow

jj commands take a long time.

**Solution:**

```bash
# For colocated repos with many branches:
# Clean up old Git branches
git branch -d old-branch
git remote prune origin

# Switch to non-colocated
# (requires re-cloning)
jj git clone <url>  # No --colocate

# Reduce log verbosity
jj config set --user revsets.log "@"
```

### Problem: High Memory Usage

jj uses too much memory.

**Solution:**

```bash
# Check repository size
du -sh .jj

# Clean up operation log
jj config set --user operation.gc-after-days 30

# For very large repos:
# Use shallow clone
# Or consider using Git directly for some operations
```

## Error Messages

### Problem: "No repo found"

```
Error: No repo found at <path>
```

**Solution:**

```bash
# Initialize repository
jj git init --colocate

# Or navigate to existing repo
cd /path/to/repo
```

### Problem: "Refusing to rebase immutable commit"

```
Error: Refusing to rebase immutable commit
```

**Solution:**

```bash
# Check immutable configuration
jj config get revsets.immutable_heads

# If you really need to rebase:
# (Not recommended for pushed commits!)
jj config set --repo revsets.immutable_heads "tags()"

# Better: Create new commit based on it
jj duplicate <immutable-commit>
jj rebase -r <duplicate> -d <destination>
```

### Problem: "Concurrent modification detected"

```
Error: Concurrent modification detected
```

**Solution:**

```bash
# Another process modified the repo
# Try again
jj status  # Re-sync

# If persists, check for:
# - Other terminals running jj
# - Background processes (IDE, git)
# - Filesystem sync issues

# Force refresh
jj op log  # Should trigger refresh
```

## Platform-Specific Issues

### Windows Issues

**Problem:** Path separators causing issues

**Solution:**
```bash
# Use forward slashes
jj log path/to/file

# Or escape backslashes
jj log "path\\to\\file"
```

**Problem:** Line ending conflicts

**Solution:**
```bash
# Configure Git to handle line endings
git config --global core.autocrlf true

# In colocated repo
jj git export
```

### macOS Issues

**Problem:** Permission denied on /usr/local

**Solution:**
```bash
# Install to user directory
cargo install --locked jj-cli

# Or use Homebrew
brew install jj
```

### Linux Issues

**Problem:** Missing shared libraries

**Solution:**
```bash
# Install dependencies
# Ubuntu/Debian:
sudo apt install libssl-dev pkg-config

# Fedora:
sudo dnf install openssl-devel

# Reinstall
cargo install --locked jj-cli --force
```

## Getting Help

### When Stuck

1. **Check documentation**
   ```bash
   jj help
   jj help <command>
   ```

2. **View operation log**
   ```bash
   jj op log  # See what happened
   ```

3. **Try undo**
   ```bash
   jj undo  # Safe to try!
   ```

4. **Check status**
   ```bash
   jj status
   jj log
   ```

### Reporting Issues

If you found a bug:

1. **Create minimal reproduction**
   ```bash
   mkdir test-repo
   cd test-repo
   jj git init --colocate
   # Minimal steps to reproduce
   ```

2. **Gather information**
   ```bash
   jj --version
   jj op log -n 5
   jj config list
   ```

3. **Report at**: https://github.com/jj-vcs/jj/issues

### Getting Community Help

- **Discord**: https://discord.gg/dkmfj3aGQN
- **GitHub Discussions**: https://github.com/jj-vcs/jj/discussions
- **IRC**: #jujutsu on Libera.Chat

## Prevention Tips

### DO

✓ Use `jj status` frequently
✓ Leverage `jj undo` when experimenting
✓ Keep operation log for recovery
✓ Test in new repo before important operations
✓ Read command help: `jj help <command>`
✓ Keep jj updated

### DON'T

✗ Force push to shared branches
✗ Delete .jj directory manually
✗ Edit .jj contents directly
✗ Panic - you can almost always recover
✗ Skip reading error messages
✗ Ignore conflicts

## Quick Diagnostic Checklist

When something goes wrong:

- [ ] Run `jj status` - does it work?
- [ ] Run `jj log` - can you see history?
- [ ] Check `jj op log` - what was last operation?
- [ ] Try `jj undo` - does it help?
- [ ] Check config: `jj config list`
- [ ] Verify in repo: `ls .jj`
- [ ] Check Git (if colocated): `git status`
- [ ] Look for error messages in full
- [ ] Try operation in test repo first

## Next Steps

- [Resources](Resources.md) - External help and learning materials
- [Best Practices](Best-Practices.md) - Avoid common issues

---

[← Back to Best Practices](Best-Practices.md) | [Main](README.md) | [Next: Resources →](Resources.md)
