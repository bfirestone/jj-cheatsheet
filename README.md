# Jujutsu (jj) Cheat Sheet

A comprehensive, modern reference for Jujutsu VCS - a powerful Git-compatible version control system that's both simple and powerful.

## What is Jujutsu?

Jujutsu (jj) is a next-generation version control system that combines the best of Git and Mercurial while fixing many pain points of traditional VCS. It's fully Git-compatible, meaning you can use it on any Git repository without migration.

### Key Features

- **Working copy as a commit** - No staging area; changes are automatically tracked
- **Change-based model** - Work with mutable changes instead of immutable commits
- **Automatic rebasing** - Descendants are automatically rebased when you modify history
- **First-class conflicts** - Conflicts don't block operations; resolve them when ready
- **Powerful undo** - Every operation can be undone with `jj undo`
- **Git compatible** - Use with any Git repository and remote

### Who Uses Jujutsu?

- Google uses it internally
- Growing open-source community
- Developers frustrated with Git's complexity

## Quick Start

```bash
# Install (see Installation.md for more options)
cargo install --git https://github.com/jj-vcs/jj jj-cli

# Initialize in an existing Git repo (colocated mode)
jj git init --colocate

# Or clone a new repository
jj git clone --colocate https://github.com/user/repo

# See what's happening
jj status
jj log

# Make changes and commit
jj new -m "Your change description"
# Edit files...
jj squash  # Move changes into the description you created

# Push to remote
jj git push
```

## 🚀 Quick Workflow Guide

**Managing multiple Git repo clones?** (e.g., `my-api_01`, `my-api_02`, ... `my-api_06`)

See **[Workflow Cheat Sheet](Cheat-Sheet-Flow.md)** to learn how Jujutsu replaces 6 repository clones with a single repo and makes feature juggling trivial. No more:

- Searching for a "free" clone
- Stashing or WIP commits to switch features
- Forgetting which clone has which feature
- Wasting disk space on duplicate repos

**One repo. Infinite parallel features. Instant context switching.**

## Documentation Structure

This repository contains comprehensive documentation organized by topic:

### Getting Started

- [Installation](Installation.md) - How to install jj on various platforms
- [Getting Started](Getting-Started.md) - Your first steps with jj
- [Core Concepts](Core-Concepts.md) - Understanding jj's mental model

### Command Reference

- [Commands Reference](Commands-Reference.md) - Complete command reference by category
- [Git Comparison](Git-Comparison.md) - Git vs jj command equivalents
- [Git Interoperability](Git-Interop.md) - Using jj with Git repositories

### Workflows & Patterns

- [Common Workflows](Workflows.md) - Daily development patterns
- [Conflict Resolution](Conflict-Resolution.md) - Handling merge conflicts
- [Advanced Features](Advanced-Features.md) - Power user features

### Configuration & Customization

- [Configuration](Configuration.md) - Config files, aliases, templates, and revsets
- [Builtin Editor](Builtin-Editor.md) - Using the interactive diff/merge editor
- [Best Practices](Best-Practices.md) - Modern best practices for 2025

### Help & Resources

- [Troubleshooting](Troubleshooting.md) - Common issues and solutions
- [Resources](Resources.md) - External tutorials and community links

## Why Switch to Jujutsu?

### From Git's Perspective

| Git Pain Point             | Jujutsu Solution                            |
| -------------------------- | ------------------------------------------- |
| Complex staging area       | No staging; working copy is always a commit |
| Confusing rebase conflicts | Conflicts don't block operations            |
| Fear of rewriting history  | Automatic rebasing + powerful undo          |
| Detached HEAD state        | Always on a change, never "detached"        |
| Lost commits               | Operation log tracks everything             |
| Merge conflict hell        | First-class conflict representation         |

### Jujutsu's Philosophy

1. **Safety first** - Operations are logged and can be undone
2. **No hidden state** - Everything is explicit and visible
3. **Flexibility** - Mutable changes until pushed
4. **Simplicity** - Consistent, predictable commands
5. **Git compatibility** - Works seamlessly with Git infrastructure

## Quick Command Reference

### Daily Workflow

```bash
jj status                    # Show working copy status
jj log                       # View commit history
jj new -m "Description"      # Start a new change
jj squash                    # Move changes into parent
jj bookmark create <name>    # Create a bookmark (branch)
jj git push                  # Push to remote
```

### Navigation

```bash
jj log                       # View history
jj show                      # Show current change
jj edit <revision>           # Switch to a different change
jj next                      # Move to child revision
jj prev                      # Move to parent revision
```

### Modification

```bash
jj describe                  # Edit current change description
jj split                     # Split change into multiple
jj squash                    # Squash change into parent
jj absorb                    # Automatically distribute changes
jj rebase -d <dest>          # Rebase to new parent
```

### Safety & Exploration

```bash
jj undo                      # Undo last operation
jj op log                    # View operation history
jj op restore <id>           # Restore to specific operation
jj abandon                   # Abandon a change
```

## Learning Path

1. **Start Here**: Read [Getting Started](Getting-Started.md) and [Core Concepts](Core-Concepts.md)
2. **If you know Git**: Review [Git Comparison](Git-Comparison.md) for command equivalents
3. **Daily Usage**: Bookmark [Commands Reference](Commands-Reference.md) and [Workflows](Workflows.md)
4. **Level Up**: Explore [Advanced Features](Advanced-Features.md) and [Best Practices](Best-Practices.md)
5. **Configure**: Customize your setup with [Configuration](Configuration.md)

## Contributing

This is a living document. Contributions, corrections, and suggestions are welcome!

## License

This documentation is provided as-is for the community. Jujutsu itself is licensed under Apache 2.0.

## Quick Links

- [Official Jujutsu Website](https://jj-vcs.github.io/jj/)
- [GitHub Repository](https://github.com/jj-vcs/jj)
- [Official Documentation](https://jj-vcs.github.io/jj/latest/)
- [Discord Community](https://discord.gg/dkmfj3aGQN)

---

**Last Updated**: 2025-10-15
**Jujutsu Version**: Latest (compatible with v0.15+)
