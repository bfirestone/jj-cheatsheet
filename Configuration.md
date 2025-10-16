# Configuration

Customize Jujutsu to match your workflow with powerful configuration options.

## Configuration Files

### File Locations

| Platform | User Config | Repo Config |
|----------|-------------|-------------|
| **Linux** | `~/.config/jj/config.toml` | `.jj/repo/config.toml` |
| **macOS** | `~/Library/Application Support/jj/config.toml` | `.jj/repo/config.toml` |
| **Windows** | `%APPDATA%\jj\config.toml` | `.jj/repo/config.toml` |

### Configuration Hierarchy

```
1. Built-in defaults
2. System config (/etc/jj/config.toml)
3. User config (~/.config/jj/config.toml)
4. Repo config (.jj/repo/config.toml)
5. Command-line --config-toml
```

Later configs override earlier ones.

## Basic Configuration

### Initial Setup

```bash
# Edit user config
jj config edit --user

# Or set values directly
jj config set --user user.name "Your Name"
jj config set --user user.email "your.email@example.com"
```

### Essential Settings

```toml
# ~/.config/jj/config.toml

[user]
name = "Your Name"
email = "your.email@example.com"

[ui]
default-command = "log"  # Command to run when no args given
editor = "vim"           # Or "code --wait", "nano", etc.
diff-editor = ":builtin" # Built-in diff editor
pager = "less -FRX"      # Pager for long output

[git]
colocate = true          # Use colocated repos by default
```

## User Interface Customization

### Editor Configuration

```toml
[ui]
# Text editor for commit messages
editor = "vim"

# Common alternatives:
# editor = "code --wait"
# editor = "subl --wait"
# editor = "emacs"
# editor = "nano"
```

### Diff Tools

```toml
[ui.diff]
tool = "difftastic"  # External diff tool

# Tool configurations
[merge-tools.difftastic]
program = "difft"
diff-args = ["$left", "$right"]
```

Popular diff tools:
- **difftastic** - Structural diff
- **delta** - Syntax-highlighted diff
- **vimdiff** - Vim-based diff
- **meld** - GUI diff viewer

### Merge Tools

```toml
[ui]
merge-editor = "meld"

[merge-tools.meld]
program = "meld"
merge-args = ["$left", "$base", "$right", "-o", "$output"]

[merge-tools.vimdiff]
program = "vim"
merge-args = ["-d", "$left", "$base", "$right", "$output"]

[merge-tools.vscode]
program = "code"
merge-args = ["--wait", "--merge", "$left", "$right", "$base", "$output"]
```

### Colors

```toml
[ui.color]
# Color settings
mode = "auto"  # auto, always, never

[colors]
"commit_id" = "blue"
"change_id" = "magenta"
"author" = "yellow"
"timestamp" = "cyan"
"bookmark" = "green"
"tag" = "green bold"
"working_copy" = "green bold"
"conflict" = "red bold"
```

## Aliases

### Command Aliases

```toml
[aliases]
# Short commands
l = ["log"]
s = ["status"]
d = ["diff"]
n = ["new"]

# Common workflows
sync = ["git", "fetch", "--all-remotes"]
push-feature = ["git", "push", "--bookmark"]

# Complex aliases
amend = ["describe"]
co = ["edit"]
br = ["bookmark", "list"]

# With arguments
log-compact = ["log", "--no-graph", "--limit", "20"]
log-verbose = ["log", "--patch"]

# Stacked changes workflow
next = ["new"]
prev = ["edit", "@-"]
```

Usage:
```bash
jj l           # Same as jj log
jj sync        # Same as jj git fetch --all-remotes
jj log-compact # Custom log view
```

### Template Aliases

```toml
[template-aliases]
# Custom timestamp format
'format_timestamp(timestamp)' = 'timestamp.ago()'

# Shorter change IDs
'format_short_change_id(id)' = 'id.shortest(8)'

# Custom author format
'format_author(author)' = 'author.name()'

# Bookmark formatting
'format_bookmarks(bookmarks)' = 'bookmarks.map(|b| b.name()).join(", ")'
```

## Revsets

### Default Revsets

```toml
[revsets]
# Default revset for jj log
log = "@ | ancestors(immutable_heads().., 2) | heads(immutable_heads())"

# Immutable commits (won't be rebased)
immutable_heads = "main@origin | tags()"

# Short log
short-log = "@ | ancestors(@, 10)"
```

### Revset Aliases

```toml
[revset-aliases]
# Your commits
'mine()' = 'author("your.email@example.com")'

# All commits by user
'user(x)' = 'author(x) | committer(x)'

# Recently changed
'recent()' = 'ancestors(@, 20)'

# Unmerged work
'unmerged()' = 'mine() & ~ancestors(main@origin)'

# Stacked changes
'stack()' = 'ancestors(@) & ~ancestors(main@origin)'

# Find bookmark
'closest_bookmark(to)' = 'heads(::to & bookmarks())'
```

Usage:
```bash
jj log -r 'mine()'      # Your commits
jj log -r 'unmerged()'  # Work not in main
jj log -r 'stack()'     # Your stack
```

## Templates

### Log Template

```toml
[templates]
# Custom log format
log = '''
separate(" ",
  change_id.shortest(8),
  if(conflict, "(conflict)"),
  if(description, description.first_line(), "(no description set)"),
  author.email(),
  timestamp.ago(),
)
'''

# Compact log
log_compact = '''
concat(
  change_id.shortest(8),
  " ",
  description.first_line(),
)
'''
```

### Advanced Templates

```toml
[templates]
# Detailed log with bookmarks
log_detailed = '''
separate(" ",
  change_id.shortest(8),
  commit_id.shortest(8),
  if(conflict, label("conflict", "(conflict)")),
  bookmarks.map(|b| b.name()).join(", "),
  if(empty, label("empty", "(empty)")),
  description.first_line(),
  label("author", author.name()),
  label("timestamp", timestamp.ago()),
)
'''

# One-line format
log_oneline = 'change_id.shortest(8) ++ " " ++ description.first_line()'
```

### Template Functions

```toml
[template-aliases]
# Commit hash with color
'colored_hash(id)' = 'label("commit_id", id.shortest(8))'

# Author name only
'short_author(author)' = 'author.name()'

# Relative timestamp
'relative_time(timestamp)' = 'timestamp.ago()'

# Bookmark list
'bookmark_names(bookmarks)' = 'bookmarks.map(|b| b.name()).join(" ")'

# Empty indicator
'show_empty(empty)' = 'if(empty, label("empty", "󰧮"), "")'
```

## Git Integration

### Git Settings

```toml
[git]
# Default to colocated repos
colocate = true

# Auto-track remote bookmarks
auto-local-bookmark = true

# Fetch settings
fetch = "origin"  # Default remote for fetch

# Push settings
push = "origin"   # Default remote for push
push-bookmark-prefix = ""  # Prefix for pushed bookmarks
```

### Remote Configuration

```toml
[git.remote.origin]
fetch = "+refs/heads/*:refs/remotes/origin/*"
push = "refs/heads/*:refs/heads/*"
```

## Advanced Configuration

### Operation Log

```toml
[operation]
# Keep operations for 180 days (default: 90)
gc-after-days = 180

# Auto-snapshot working copy
auto-snapshot = true
```

### Signature Settings

```toml
[signature]
# Sign commits
sign-all = false

# GPG key
key = "your-gpg-key-id"

# Sign with SSH
backend = "ssh"
key = "~/.ssh/id_ed25519.pub"
```

### Formatting

```toml
[format]
# Wrap long lines
wrap = 80

# Tree style
tree-level-indent = "  "

# Graph style
graph-style = "curved"  # or "square", "ascii"
```

### Snapshot Settings

```toml
[snapshot]
# Snapshot working copy before operations
auto-snapshot = true

# Maximum snapshot age
max-age = "1 hour"
```

## Per-Repository Configuration

### Repo-Specific Settings

```bash
# Edit repo config
cd your-repo
jj config edit --repo
```

```toml
# .jj/repo/config.toml

[revsets]
# This repo has different main branch
immutable_heads = "develop@origin | tags()"

[aliases]
# Repo-specific aliases
deploy = ["bookmark", "create", "deploy-candidate"]
```

### Shared Team Config

Commit `.jj/repo/config.toml` to share with team:

```toml
# .jj/repo/config.toml (checked into Git)

[revsets]
log = "@ | ancestors(main@origin.., 2) | heads(main@origin)"
immutable_heads = "main@origin | stable@origin | tags()"

[template-aliases]
'format_timestamp(timestamp)' = 'timestamp.format("%Y-%m-%d")'
```

## Example Configurations

### Minimal Configuration

```toml
# ~/.config/jj/config.toml
[user]
name = "Your Name"
email = "your.email@example.com"

[ui]
editor = "vim"
default-command = "log"

[git]
colocate = true
```

### Power User Configuration

```toml
# ~/.config/jj/config.toml
[user]
name = "Your Name"
email = "your.email@example.com"

[ui]
editor = "code --wait"
default-command = "log"
diff-editor = ":builtin"
pager = "delta"

[ui.diff]
tool = "difftastic"

[merge-tools.difftastic]
program = "difft"

[ui.merge-editor]
tool = "meld"

[merge-tools.meld]
program = "meld"

[git]
colocate = true
auto-local-bookmark = true

[aliases]
l = ["log"]
ll = ["log", "--patch"]
s = ["status"]
d = ["diff"]
ds = ["diff", "--stat"]
n = ["new"]
sync = ["git", "fetch", "--all-remotes"]
rb = ["rebase", "-d", "main@origin"]

[revset-aliases]
'mine()' = 'author("your.email@example.com")'
'stack()' = 'ancestors(@) & ~ancestors(main@origin)'
'recent()' = 'ancestors(all(), 20)'

[revsets]
log = "@ | ancestors(immutable_heads().., 5) | heads(immutable_heads())"
immutable_heads = "main@origin | tags()"

[template-aliases]
'format_timestamp(timestamp)' = 'timestamp.ago()'
'format_short_change_id(id)' = 'id.shortest(8)'

[colors]
"commit_id" = "blue"
"change_id" = "magenta"
"author" = "yellow"
"working_copy" = "green bold"
"conflict" = "red bold"
```

### Team Configuration

```toml
# .jj/repo/config.toml (shared in repo)
[revsets]
log = "@ | ancestors(main@origin.., 3) | heads(main@origin)"
immutable_heads = "main@origin | release@origin | tags()"

[template-aliases]
'format_timestamp(timestamp)' = 'timestamp.format("%Y-%m-%d %H:%M")'

[aliases]
deploy = ["bookmark", "create", "deploy-candidate"]
test-all = ["!cargo", "test", "--all"]
```

## Configuration Tips

### View Current Config

```bash
# Show all config
jj config list

# Show specific value
jj config get user.name

# Show config with sources
jj config list --include-defaults
```

### Override on Command Line

```bash
# Temporary override
jj --config-toml='ui.editor="nano"' describe

# Multiple overrides
jj --config-toml='ui.color.mode="always"' \
   --config-toml='ui.pager="cat"' \
   log
```

### Testing Config Changes

```bash
# Edit config
jj config edit --user

# Test immediately
jj log  # See if changes work

# If broken, fix it
jj config edit --user
```

### Backup Config

```bash
# Backup your config
cp ~/.config/jj/config.toml ~/.config/jj/config.toml.backup

# Restore if needed
cp ~/.config/jj/config.toml.backup ~/.config/jj/config.toml
```

## Common Customizations

### Show Change IDs in Uppercase

```toml
[template-aliases]
'format_short_change_id(id)' = 'id.shortest(8).upper()'
```

### Custom Log with Bookmarks

```toml
[templates]
log = '''
concat(
  change_id.shortest(8),
  " ",
  if(bookmarks, concat("[", bookmarks.join(", "), "] ")),
  description.first_line(),
  " -- ",
  author.name(),
  " ",
  timestamp.ago(),
)
'''
```

### Git-like Aliases

```toml
[aliases]
commit = ["describe"]
checkout = ["edit"]
branch = ["bookmark"]
pull = ["git", "fetch"]
push = ["git", "push"]
log-oneline = ["log", "--no-graph"]
```

## Troubleshooting Config

### Config Not Loading

```bash
# Check file location
jj config path

# Check syntax
jj config list  # Should show error if invalid TOML
```

### Syntax Errors

```toml
# Wrong:
[user
name = "Test"

# Correct:
[user]
name = "Test"
```

### Verify Changes

```bash
# After editing config
jj config get user.name  # Should show new value
jj config list           # Should show all settings
```

## Next Steps

- [Advanced Features](Advanced-Features.md) - Use custom config for power features
- [Builtin Editor](Builtin-Editor.md) - Configure diff editor
- [Best Practices](Best-Practices.md) - Recommended configurations

---

[← Back to Conflict Resolution](Conflict-Resolution.md) | [Main](README.md) | [Next: Advanced Features →](Advanced-Features.md)
