# Installing Jujutsu

Complete installation guide for Jujutsu (jj) on all major platforms.

## Quick Install

### macOS

```bash
# Using Homebrew
brew install jj

# Or using MacPorts
sudo port install jj
```

### Linux

#### Arch Linux

```bash
# Using pacman
sudo pacman -S jujutsu

# Or from AUR
yay -S jujutsu-git
```

#### Fedora/RHEL

```bash
# Fedora 39+
sudo dnf install jj

# Or using Copr
sudo dnf copr enable walters/jujutsu
sudo dnf install jujutsu
```

#### Ubuntu/Debian

```bash
# Add the repository (for Ubuntu 22.04+)
sudo add-apt-repository ppa:jujutsu/jj
sudo apt update
sudo apt install jj

# Or download .deb from GitHub releases
wget https://github.com/jj-vcs/jj/releases/latest/download/jj_*_amd64.deb
sudo dpkg -i jj_*_amd64.deb
```

#### NixOS

```bash
# Temporary shell
nix-shell -p jujutsu

# Or add to configuration.nix
environment.systemPackages = [ pkgs.jujutsu ];
```

#### Gentoo

```bash
emerge -av dev-vcs/jujutsu
```

### Windows

```powershell
# Using Scoop
scoop install jj

# Or using Winget
winget install jj

# Or using Chocolatey
choco install jujutsu
```

### FreeBSD

```bash
pkg install jujutsu
```

## Install from Source

### Using Cargo (Rust)

This works on all platforms with Rust installed:

```bash
# Install latest released version
cargo install --locked jj-cli

# Or install from Git (development version)
cargo install --git https://github.com/jj-vcs/jj jj-cli --locked
```

### Prerequisites for Building from Source

```bash
# Install Rust if not already installed
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# macOS: Install additional dependencies
brew install pkg-config openssl

# Ubuntu/Debian: Install additional dependencies
sudo apt install build-essential pkg-config libssl-dev

# Fedora: Install additional dependencies
sudo dnf install gcc pkg-config openssl-devel
```

### Building from Source

```bash
# Clone the repository
git clone https://github.com/jj-vcs/jj.git
cd jj

# Build and install
cargo install --path cli --locked

# Or just build without installing
cargo build --release
# Binary will be at target/release/jj
```

## Verify Installation

```bash
# Check version
jj --version

# Should output something like:
# jj 0.24.0

# Test basic functionality
jj --help
```

## Shell Completion (Optional but Recommended)

### Bash

```bash
# Generate completion script
jj util completion bash > ~/.jj-completion.bash

# Add to ~/.bashrc
echo 'source ~/.jj-completion.bash' >> ~/.bashrc
```

### Zsh

```bash
# Generate completion script
jj util completion zsh > ~/.zsh/completion/_jj

# Make sure the directory is in your fpath (add to ~/.zshrc if needed)
fpath=(~/.zsh/completion $fpath)
autoload -Uz compinit && compinit
```

### Fish

```bash
# Generate completion script
jj util completion fish > ~/.config/fish/completions/jj.fish
```

### PowerShell (Windows)

```powershell
# Generate completion script
jj util completion powershell > $PROFILE\..\jj-completion.ps1

# Add to your PowerShell profile
Add-Content $PROFILE "`n. `$PSScriptRoot\jj-completion.ps1"
```

## Initial Configuration

After installation, set up your identity:

```bash
# Set your name and email
jj config set --user user.name "Your Name"
jj config set --user user.email "your.email@example.com"

# Or edit config file directly
jj config edit --user
```

The config file will be created at:
- Linux: `~/.config/jj/config.toml`
- macOS: `~/Library/Application Support/jj/config.toml`
- Windows: `%APPDATA%\jj\config.toml`

### Minimal Config Example

```toml
[user]
name = "Your Name"
email = "your.email@example.com"

[ui]
default-command = "log"
diff-editor = ":builtin"
```

## Editor Configuration

Jujutsu uses your system's default editor. To set a specific editor:

```bash
# Set editor in jj config
jj config set --user ui.editor "vim"

# Or use environment variable
export EDITOR=vim  # Add to ~/.bashrc or ~/.zshrc
```

Popular editor options:
```bash
# VS Code
jj config set --user ui.editor "code --wait"

# Sublime Text
jj config set --user ui.editor "subl --wait"

# Emacs
jj config set --user ui.editor "emacs"

# Nano
jj config set --user ui.editor "nano"

# Neovim
jj config set --user ui.editor "nvim"
```

## Git Integration

If you plan to use jj with Git repositories, ensure Git is installed:

```bash
# Check if git is installed
git --version

# If not installed:
# macOS
brew install git

# Ubuntu/Debian
sudo apt install git

# Fedora
sudo dnf install git

# Windows
winget install Git.Git
```

## Optional Tools

### Diff Tools

Configure a diff tool for better visualization:

```bash
# Configure external diff tool
jj config set --user ui.diff.tool "difftastic"

# Popular diff tools:
# - difftastic (recommended)
# - delta
# - vimdiff
# - meld
```

Install difftastic:
```bash
# macOS
brew install difftastic

# Cargo (all platforms)
cargo install difftastic

# Arch Linux
sudo pacman -S difftastic
```

### Merge Tools

```bash
# Configure merge tool
jj config set --user merge-tools.meld.program "meld"

# Popular merge tools:
# - meld
# - kdiff3
# - vimdiff
# - Beyond Compare (bcompare)
```

## Docker Installation

Run jj in a Docker container:

```dockerfile
FROM rust:latest
RUN cargo install --locked jj-cli
WORKDIR /workspace
CMD ["jj"]
```

```bash
# Build and run
docker build -t jj-container .
docker run -it -v $(pwd):/workspace jj-container jj status
```

## Upgrading Jujutsu

### Package Managers

```bash
# macOS
brew upgrade jj

# Arch Linux
sudo pacman -Syu jujutsu

# Ubuntu (PPA)
sudo apt update && sudo apt upgrade jj

# Windows (Scoop)
scoop update jj
```

### Cargo

```bash
# Upgrade to latest release
cargo install --locked jj-cli --force

# Or upgrade to latest development version
cargo install --git https://github.com/jj-vcs/jj jj-cli --locked --force
```

## Uninstalling

### Package Managers

```bash
# macOS
brew uninstall jj

# Arch Linux
sudo pacman -R jujutsu

# Ubuntu
sudo apt remove jj

# Windows (Scoop)
scoop uninstall jj
```

### Cargo

```bash
cargo uninstall jj-cli
```

### Remove Configuration

```bash
# Linux
rm -rf ~/.config/jj

# macOS
rm -rf ~/Library/Application\ Support/jj

# Windows
rmdir /s %APPDATA%\jj
```

## Troubleshooting Installation

### Cargo Build Fails

```bash
# Update Rust
rustup update stable

# Clean and rebuild
cargo clean
cargo build --release
```

### Command Not Found After Installation

```bash
# Check if cargo bin is in PATH
echo $PATH | grep cargo

# Add cargo bin to PATH (add to ~/.bashrc or ~/.zshrc)
export PATH="$HOME/.cargo/bin:$PATH"

# Reload shell
source ~/.bashrc  # or source ~/.zshrc
```

### Permission Denied

```bash
# Linux/macOS: Ensure binary is executable
chmod +x $(which jj)

# Or reinstall with proper permissions
cargo install --locked jj-cli --force
```

## Next Steps

After installation:

1. [Getting Started](Getting-Started.md) - Initialize your first repository
2. [Core Concepts](Core-Concepts.md) - Understand jj's mental model
3. [Configuration](Configuration.md) - Customize your setup

---

[← Back to Main](README.md) | [Next: Getting Started →](Getting-Started.md)
