# dotfiles

Personal dotfiles managed with [chezmoi](https://chezmoi.io). Bootstraps a fresh machine from zero to fully configured in one command — packages, shell, tools, and encrypted secrets all included.

Works on **macOS** (primary) and **Linux** (Debian/Ubuntu and Arch). All OS-specific logic lives in chezmoi templates — no runtime branching in shell configs.

## Table of Contents

- [Key Features](#key-features)
- [Tech Stack](#tech-stack)
- [What Gets Managed](#what-gets-managed)
- [Prerequisites](#prerequisites)
- [Bootstrap](#bootstrap)
  - [macOS](#macos)
  - [Linux](#linux)
  - [Answer the Setup Prompts](#answer-the-setup-prompts)
  - [Verify the Setup](#verify-the-setup)
  - [Post-Bootstrap Steps](#post-bootstrap-steps-manual)
- [Architecture](#architecture)
- [Directory Structure](#directory-structure)
- [Template Variables](#template-variables)
- [Encryption](#encryption)
- [Day-to-Day Usage](#day-to-day-usage)
- [Shell Setup](#shell-setup)
- [Docker & Home Assistant (Linux)](#docker--home-assistant-linux)
- [Installed Tools](#installed-tools)
- [Adding New Files](#adding-new-files)
- [Troubleshooting](#troubleshooting)

---

## Key Features

- **One-command bootstrap** — `chezmoi init --apply larham/linux-dotfiles` sets up a new machine end-to-end
- **Cross-platform** — macOS (Apple Silicon) and Linux (Debian/Ubuntu, Arch) with OS-gated templates
- **Age encryption** support for secrets (gracefully skipped if `~/.config/chezmoi/key.txt` is not present)
- **Templated configs** — shell files and git config adapt based on OS, user identity, GPG key, and whether OrbStack is installed
- **Optional GPG commit signing** — prompted during init; signs commits automatically when a key ID is provided
- **Auto-installs packages** — Homebrew + `Brewfile` on macOS; `apt`/`pacman` + official repos on Linux
- **Linux desktop & server polish** — macOS-style `Cmd` keybindings (`keyd` + GNOME), native Firefox `.deb` & Google Chrome, Ghostty terminal, sleep/lid-switch disabled for always-on server use
- **Home Assistant on Docker (Linux)** — automatically installs Docker Engine + Compose V2, deploys Home Assistant with HACS and USB Zigbee pass-through in `~/home-assistant/`
- **Zsh on all platforms** — installed and set as default shell on Linux automatically
- **Conventional commit template** enforced globally for all git repos

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| [chezmoi](https://chezmoi.io) | Dotfile manager |
| [age](https://age-encryption.org) | Secret encryption |
| [Homebrew](https://brew.sh) | Package manager (macOS only) |
| [Oh My Zsh](https://ohmyz.sh) | Zsh framework |
| [Starship](https://starship.rs) | Shell prompt |
| [fzf](https://github.com/junegunn/fzf) | Fuzzy finder |
| [fnm](https://github.com/Schniz/fnm) | Node.js version manager |
| [direnv](https://direnv.net) | Per-directory environment loader |
| [zoxide](https://github.com/ajeetdsouza/zoxide) | Smart `cd` |
| [eza](https://github.com/eza-community/eza) | Modern `ls` |
| [bat](https://github.com/sharkdp/bat) | Modern `cat` |
| [yazi](https://yazi-rs.github.io) | Terminal file manager |
| [Ghostty](https://ghostty.org) | Terminal emulator (macOS & Linux) |
| [Docker Compose](https://docs.docker.com/compose/) | Container runtime for Home Assistant (Linux) |
| [Home Assistant](https://www.home-assistant.io) | Smart home automation server + HACS (Linux) |

---

## What Gets Managed

| Path | chezmoi source | Notes |
|------|---------------|-------|
| `~/.zshrc` | `dot_zshrc` | Delegates to `~/.config/zsh/.zshrc` |
| `~/.zprofile` | `dot_zprofile.tmpl` | Templated (Homebrew/OrbStack — macOS only) |
| `~/.config/zsh/` | `dot_config/zsh/` | All zsh config modules (templated) |
| `~/.config/git/config` | `dot_config/git/config.tmpl` | Templated (name, email, optional GPG signing) |
| `~/.config/git/ignore` | `dot_config/git/ignore` | Global gitignore |
| `~/.config/git/template` | `dot_config/git/template` | Commit message template |
| `~/.config/gh/config.yml` | `dot_config/gh/private_config.yml` | GitHub CLI preferences |
| `~/.config/ghostty/` | `dot_config/ghostty/` | Terminal config, themes, icons |
| `~/.config/starship/` | `dot_config/starship/` | Prompt config |
| `~/.config/bat/` | `dot_config/bat/` | `cat` replacement config |
| `~/.config/eza/` | `dot_config/eza/` | `ls` replacement config |
| `~/.config/yazi/` | `dot_config/yazi/` | File manager config |
| `~/.config/fastfetch/` | `dot_config/fastfetch/` | System info display |
| `~/.config/btop/` | `dot_config/btop/` | Resource monitor config |
| `~/home-assistant/` | `home-assistant/` | Home Assistant `compose.yaml` & `config/configuration.yaml` (Linux only) |
| `~/Brewfile` (applied at source) | `Brewfile` | All Homebrew packages and casks (macOS only) |

**Not tracked** (excluded via `.chezmoiignore`):
- `~/.config/opencode/` — AI coding assistant (machine-specific)
- `~/.config/nvim/` — Neovim config (managed separately)
- `~/.config/op/` — 1Password CLI (machine-specific)
- `~/home-assistant/` on macOS — skipped automatically when `.chezmoi.os != "linux"`
- Age-encrypted files (`*.age`) when `~/.config/chezmoi/key.txt` is not present

---

## Prerequisites

### macOS

- **macOS** (Apple Silicon assumed; Homebrew path `/opt/homebrew`)
- **Internet access** for Homebrew and package installation
- **(Optional) age key file** at `~/.config/chezmoi/key.txt` to decrypt age-encrypted secrets (if missing, `.chezmoiignore` automatically skips encrypted files):
  ```bash
  mkdir -p ~/.config/chezmoi
  # Paste your age private key into:
  nano ~/.config/chezmoi/key.txt
  chmod 600 ~/.config/chezmoi/key.txt
  ```

### Linux

- **Debian/Ubuntu** or **Arch Linux**
- **Internet access** for package installation
- **`sudo` access** for package manager commands, system services, and setting default shell
- **(Optional) age key file** at `~/.config/chezmoi/key.txt` (same as macOS)
- `curl` must be available before bootstrapping (pre-installed on most distros)

---

## Bootstrap

### macOS

```bash
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply larham/linux-dotfiles
```

This single command:
1. Downloads and installs chezmoi
2. Clones `github.com/larham/linux-dotfiles` to `~/.local/share/chezmoi`
3. Runs `run_once_before_10-install-homebrew.sh.tmpl` — installs Homebrew if absent
4. Runs `run_once_before_20-install-packages.sh.tmpl` — runs `brew bundle` from `Brewfile`, then installs oh-my-zsh and its plugins
5. Prompts for configuration values (see below)
6. Applies all dotfiles to their target locations
7. Sets AC display sleep timeout to 20 minutes via `pmset`

### Linux

```bash
sh -c "$(curl -fsLS get.chezmoi.io)" -- -b ~/.local/bin init --apply larham/linux-dotfiles
```

On Linux, the bootstrap process:
1. Installs chezmoi to `~/.local/bin`
2. Detects the distro via `/etc/os-release`
3. **Arch**: runs `pacman -Syu` and installs all packages in one shot, plus `fnm`, `oh-my-zsh`, and plugins
4. **Debian/Ubuntu**: runs `apt-get install` for base packages, configures native APT repos/PPAs for `gh`, `firefox` (replacing Snap), `google-chrome-stable`, `fastfetch`, and `ghostty`, and installs `fzf`, `starship`, `zoxide`, `lazygit`, `chezmoi`, `just`, `sops`, `fnm`, `oh-my-zsh`, and its plugins
5. Sets zsh as the default shell (`chsh`) if not already
6. Applies all dotfiles (including `~/home-assistant/compose.yaml` and `~/home-assistant/config/configuration.yaml`)
7. Configures always-on server power settings (disables system sleep/suspend/lid switch, enables `ssh` and `tlp`), GNOME desktop favorites & notification settings, macOS-style `Cmd` keybindings (`keyd` + GNOME XKB), and deploys Home Assistant with HACS in Docker

> **Note:** Homebrew is **not** installed on Linux. Native package managers are used instead.

### Answer the Setup Prompts

chezmoi will ask questions on first apply:

| Prompt | Description | Stored as |
|--------|-------------|-----------|
| `Email address` | Your git/personal email | `data.email` |
| `Full name` | Your full name | `data.name` |
| `GPG key ID for git signing (leave empty to disable)` | Optional GPG key ID — enables `commit.gpgsign = true` in git config | `data.gpgKeyId` |
| `Is OrbStack installed?` | Enables OrbStack shell integration in `~/.zprofile` (macOS only) | `data.hasOrbStack` |

Answers are saved to `~/.config/chezmoi/chezmoi.toml` and will not be asked again.

### Verify the Setup

```bash
# Check shell is working
fastfetch

# Check git config was applied
git config --global user.email
git config --global user.name

# Check GPG signing (if configured)
git config --global commit.gpgsign

# (Linux) Check Home Assistant container is running
docker ps
```

### Post-Bootstrap Steps (Manual)

These are not automated and must be done manually after bootstrapping:

- **GitHub CLI auth** — Authenticate with GitHub:
  ```bash
  gh auth login
  ```
- **(Linux) Log out and back in** — Required for the default shell (`zsh`) and `docker`/`dialout`/`tty` group memberships to take effect in new terminal sessions.
- **(macOS only) 1Password SSH agent** — If using 1Password for SSH keys, enable in 1Password settings → Developer → SSH Agent.

---

## Architecture

### How chezmoi Works

chezmoi stores a *source* copy of every dotfile in `~/.local/share/chezmoi/`. When you run `chezmoi apply`, it compares the source to the target (your home directory) and copies any changed files.

**chezmoi is NOT symlink-based** — it physically copies files. The source directory is the single source of truth.

### Naming Conventions

chezmoi uses filename prefixes/suffixes to control behavior:

| Prefix/suffix | Meaning |
|--------------|---------|
| `dot_` | Becomes `.` in target (e.g., `dot_zshrc` → `~/.zshrc`) |
| `private_` | Applies `0600` permissions in target |
| `.tmpl` suffix | Rendered as a Go template before applying |
| `.age` suffix | Decrypted with age before applying |
| `run_before_` | Shell script run on every `chezmoi apply`, before applying dotfiles |
| `run_once_before_` | Shell script run once (tracked by content hash), before applying dotfiles |
| `run_once_after_` | Shell script run once (tracked by content hash), after applying dotfiles |
| `run_onchange_after_` | Shell script run whenever its rendered content changes, after applying dotfiles |
| `encrypted_` | Convention (not enforced) — file is age-encrypted |

### Bootstrap Script Order

chezmoi executes `before_` scripts prior to writing target files, and `after_` scripts once target files are in place (ordered lexicographically):

**Before applying files:**
1. `run_once_before_10-install-homebrew.sh.tmpl` — Installs Homebrew (macOS only; no-op on Linux)
2. `run_once_before_20-install-packages.sh.tmpl` — Installs packages (`brew bundle` on macOS; `apt`/`pacman` + official APT repos/PPAs/install scripts on Linux) and Oh My Zsh + plugins; sets `zsh` as default shell on Linux
3. `run_before_50-chown-homeassistant.sh.tmpl` — Checks if `~/home-assistant/config` contains `root`-owned files created by the Home Assistant container and runs `sudo chown -R $USER:$USER` so chezmoi can cleanly update config files (Linux only)

**After applying files:**
4. `run_onchange_after_25-install-firefox-apt.sh.tmpl` — Replaces the Ubuntu Firefox Snap with the native `.deb` from Mozilla's official APT repo and configures clean startup/new-tab policies (Linux only)
5. `run_onchange_after_26-install-chrome-apt.sh.tmpl` — Installs `google-chrome-stable` from Google's official APT repo (Linux only)
6. `run_once_after_30-disable-sleep.sh.tmpl` — Masks systemd sleep/suspend/hibernate targets, ignores laptop lid switch, enables `ssh`/`sshd`, and enables `tlp` power management (Linux only)
7. `run_onchange_after_40-gnome-favorites.sh.tmpl` — Pins Ghostty, Firefox, and GNOME Settings to the GNOME dock; silences update notification popups (Snap, App Center, update-notifier, Firefox); sets a 20-minute screen blank timeout (`idle-delay 1200` on Linux, `pmset -c displaysleep 20` on macOS) with AC auto-suspend disabled; opens UFW ports `8123/tcp` (Home Assistant) and `22/tcp` (SSH)
8. `run_onchange_after_50-macos-keymap.sh.tmpl` — Swaps Left Command and Left Control via GNOME XKB options and kernel-level `keyd` for system-wide macOS-style `Cmd+C`/`Cmd+V` shortcuts, disables `Super+V` notification tray binding, and maps `Command+Space` to GNOME Overview search (Linux only)
9. `run_once_after_60-install-homeassistant-docker.sh.tmpl` — Installs Docker Engine and the Docker Compose V2 plugin, adds `$USER` to `docker`/`dialout`/`tty` groups, disables `ModemManager`, opens UFW ports `8123` and `22`, starts Home Assistant via Docker Compose in `~/home-assistant/`, and installs HACS (Linux only)

`run_once_` scripts only execute once per machine (chezmoi tracks them by content hash), while `run_onchange_` scripts re-run automatically whenever their contents change.

### OS Branching

All platform-specific logic lives in chezmoi templates using `.chezmoi.os` and `.chezmoi.osRelease.id`:

```go
{{- if eq .chezmoi.os "darwin" }}
# macOS-only block
{{- else if eq .chezmoi.os "linux" }}
{{-   if eq .chezmoi.osRelease.id "arch" }}
# Arch-only block
{{-   else }}
# Debian/Ubuntu block
{{-   end }}
{{- end }}
```

No runtime OS detection happens in any shell config file.

---

## Directory Structure

```
~/.local/share/chezmoi/
├── .chezmoi.toml.tmpl               # Config template (prompts, encryption settings)
├── .chezmoiignore                   # Files excluded from management (OS & key gated)
├── .gitignore                       # Excludes age key and other local-only files
├── Brewfile                         # All Homebrew packages and casks (macOS only)
├── dot_zshrc                        # → ~/.zshrc (delegates to ~/.config/zsh/.zshrc)
├── dot_zprofile.tmpl                # → ~/.zprofile (Homebrew/OrbStack — macOS gated)
├── run_once_before_10-install-homebrew.sh.tmpl        # Installs Homebrew (macOS only)
├── run_once_before_20-install-packages.sh.tmpl        # Installs packages (cross-platform)
├── run_before_50-chown-homeassistant.sh.tmpl          # Fixes root ownership in ~/home-assistant/config (Linux only)
├── run_onchange_after_25-install-firefox-apt.sh.tmpl  # Native Firefox .deb & policies via Mozilla APT (Linux only)
├── run_onchange_after_26-install-chrome-apt.sh.tmpl   # Installs google-chrome-stable via Google APT (Linux only)
├── run_once_after_30-disable-sleep.sh.tmpl            # Disables sleep, enables SSH & TLP (Linux only)
├── run_onchange_after_40-gnome-favorites.sh.tmpl      # GNOME dock favorites, notifications, 20m screen timeout, UFW
├── run_onchange_after_50-macos-keymap.sh.tmpl         # macOS Cmd keybindings via keyd & GNOME XKB (Linux only)
├── run_once_after_60-install-homeassistant-docker.sh.tmpl # Installs Docker & deploys Home Assistant + HACS (Linux only)
├── home-assistant/                  # → ~/home-assistant/ (Linux only; ignored on macOS)
│   ├── compose.yaml                 # Docker Compose service definition for Home Assistant
│   └── config/
│       └── configuration.yaml       # Base Home Assistant configuration
├── dot_config/
│   ├── zsh/
│   │   ├── dot_zshrc                # → ~/.config/zsh/.zshrc (sources all modules)
│   │   ├── aliases.zsh.tmpl         # Shell aliases (macOS/Linux variants)
│   │   ├── env.zsh.tmpl             # $PATH, $EDITOR, Starship, zoxide, fnm, direnv init
│   │   ├── fzf.zsh                  # fzf + yazi shell integration
│   │   ├── history.zsh              # Zsh history settings
│   │   └── omz.zsh.tmpl             # Oh My Zsh config (macos plugin gated)
│   ├── git/
│   │   ├── config.tmpl              # → ~/.config/git/config (name, email, optional GPG signing)
│   │   ├── ignore                   # → ~/.config/git/ignore (global gitignore)
│   │   └── template                 # → ~/.config/git/template (commit message template)
│   ├── gh/
│   │   └── private_config.yml       # → ~/.config/gh/config.yml
│   ├── ghostty/
│   │   ├── config                   # Terminal settings (font, theme, keybinds, copy-on-select)
│   │   ├── icons/                   # Custom app icon
│   │   └── themes/                  # Custom color themes
│   ├── starship/
│   │   └── starship.toml            # Prompt configuration
│   ├── bat/                         # bat (cat replacement) config
│   ├── eza/                         # eza (ls replacement) config
│   ├── yazi/
│   │   ├── yazi.toml                # File manager config
│   │   ├── keymaps.toml             # Custom keybindings
│   │   ├── theme.toml               # Color theme
│   │   ├── package.toml             # yazi plugins
│   │   └── flavors/                 # Theme flavors (ayu-dark, catppuccin-*, dracula, flexoki, kanagawa)
│   ├── fastfetch/                   # System info display config
│   └── btop/                        # Resource monitor config
└── private_dot_ssh/                 # → ~/.ssh/ (directory for SSH configs)
```

---

## Template Variables

The following data variables are available in all `.tmpl` files:

| Variable | Type | Description |
|----------|------|-------------|
| `.chezmoi.os` | string | `"darwin"` or `"linux"` |
| `.chezmoi.osRelease.id` | string | `"arch"`, `"debian"`, `"ubuntu"`, etc. (Linux only) |
| `.chezmoi.sourceDir` | string | Absolute path to the chezmoi source directory |
| `.email` | string | User's email address (from prompt) |
| `.name` | string | User's full name (from prompt) |
| `.gpgKeyId` | string | GPG key ID for git commit signing (empty string disables signing) |
| `.hasOrbStack` | bool | Whether OrbStack is installed (macOS only; always `false` on Linux) |

Example usage in a template:

```toml
[user]
  email = "{{ .email }}"
  name = "{{ .name }}"
{{- if .gpgKeyId }}
  signingKey = "{{ .gpgKeyId }}"
{{- end }}
```

---

## Encryption

Secret encryption is configured with [age](https://age-encryption.org) in `.chezmoi.toml.tmpl`.

### Key Details

- **age key location**: `~/.config/chezmoi/key.txt` (never committed, `chmod 600`)
- **Public recipient**: `age16mv7ycglyrn68pnrfv6r2pf72swjffn20sdehfdha7l5sqlr3s2qa2mlsl`
- **Fallback behavior**: If `~/.config/chezmoi/key.txt` is not present on the machine, `.chezmoiignore` automatically ignores age-encrypted files (`dot_config/gh/encrypted_private_hosts.yml.age` and `private_dot_ssh/encrypted_config.age`) so `chezmoi apply` still succeeds without errors.

### Encrypting a New Secret File

```bash
# Encrypt a file and add it to chezmoi
chezmoi add --encrypt ~/.config/some/secret-file

# Or encrypt inline during edit
chezmoi encrypt ~/.config/some/plaintext-file > \
  ~/.local/share/chezmoi/dot_config/some/encrypted_secret-file.age
```

### Editing an Encrypted File

```bash
# Opens decrypted content in $EDITOR, re-encrypts on save
chezmoi edit ~/.config/some/secret-file
```

---

## Day-to-Day Usage

### Edit a Tracked File

```bash
# Opens the source file in $EDITOR and applies on save
chezmoi edit ~/.zshrc

# Edit and immediately apply
chezmoi edit --apply ~/.zshrc
```

### Apply Changes

```bash
# Apply all pending changes
chezmoi apply

# Preview what would change (dry run)
chezmoi diff

# Apply a single file
chezmoi apply ~/.zshrc
```

### Check Status

```bash
# Show which files differ between source and target
chezmoi status
```

### Commit Changes

```bash
# Navigate to the source repo
cd ~/.local/share/chezmoi

# Commit all changes
git add -A && git commit -m "chore: update aliases"
git push
```

Or use the shortcut alias:

```bash
cm cd   # chezmoi cd — jumps to source directory
```

### Pull Changes on Another Machine

```bash
# Pull and apply latest changes from GitHub
chezmoi update
```

### Re-Run a Bootstrap Script

`run_once_` scripts run only once (tracked by content hash). To force re-run:

```bash
# Reset the run-once state for all scripts
chezmoi state delete-bucket --bucket=scriptState
chezmoi apply
```

Or edit the script slightly (e.g., add a comment) to change its hash.

---

## Shell Setup

The Zsh configuration is split into focused modules under `~/.config/zsh/`:

### Module Overview

| File | Purpose |
|------|---------|
| `env.zsh.tmpl` | `$PATH` (`~/.local/bin`, Bun, plus Homebrew & Antigravity on macOS), `$EDITOR` (`nvim`), `en_US.UTF-8` locale, Starship, zoxide, fnm, and direnv init |
| `history.zsh` | History file size (1,000,000 entries), dedup and sharing options |
| `fzf.zsh` | fzf key bindings (`source <(fzf --zsh)`), previews using `bat` and `eza`, yazi `y()` wrapper |
| `omz.zsh.tmpl` | Oh My Zsh theme (none — Starship handles prompt), plugins list |
| `aliases.zsh.tmpl` | All shell aliases (OS-specific variants) |
| `.zshrc` | Sources all modules in order |

### Notable Aliases

| Alias | Command | Description |
|-------|---------|-------------|
| `ls` | `eza --color=always --icons=always --long --header --git --time-style=relative --group-directories-first` | Long listing with git status and relative timestamps |
| `lt` | `eza --tree --level=2 --long --icons --git` | Tree view (2 levels, with sizes) |
| `ltree` | `eza --tree --level=2 --icons --git` | Tree view (2 levels, compact) |
| `cd` | `z` | zoxide smart directory jump |
| `ff` | `fastfetch` | System info |
| `sysup` | macOS: `brew upgrade; mas upgrade; omz update` / Arch: `sudo pacman -Syu; omz update` / Debian: `sudo apt update && sudo apt upgrade -y; omz update` | Update everything |
| `buc` | `brew upgrade --cask` | Upgrade Homebrew casks only (macOS) |
| `opc` | `opencode` | AI coding assistant |
| `cm` | `chezmoi` | chezmoi shortcut |
| `gh-create` | `gh repo create --private --source=. --remote=origin && git push -u --all && gh browse` | Create private GitHub repo and push |
| `fh` | `fc -l 1 \| tac \| fzf --no-sort --border` | Fuzzy search shell history |
| `rr` | `source ~/.zshrc` | Reload shell config |
| `c` / `e` | `clear` / `exit` | Clear terminal / exit shell |
| `shutdown` / `reboot` | `sudo shutdown now` / `sudo reboot` | Power off / reboot machine |
| `doc` / `dow` | `$HOME/Documents` / `$HOME/Downloads` | Quick path shortcuts |
| `ql` | `qlmanage -p` | Quick Look preview (macOS only) |
| `spoon` | `/Applications/data-integration/spoon.sh` | Launch Pentaho Data Integration (macOS only) |
| `idea` | `$HOME/Applications/IntelliJ\ IDEA\ Ultimate.app/Contents/MacOS/idea` | Launch IntelliJ IDEA Ultimate (macOS only) |

### Oh My Zsh Plugins

```
git                      # Git aliases and prompt info
sudo                     # Double Esc to prepend sudo
zsh-autosuggestions      # Fish-style autosuggestions
zsh-syntax-highlighting  # Syntax highlighting as you type
web-search               # web-search google <query>
copyfile                 # Copy file contents to clipboard
copybuffer               # Ctrl+O copies current buffer
dirhistory               # Alt+Arrow for directory history
python                   # Python virtualenv helpers
history                  # History aliases (h, hs, hsi)
macos                    # macOS-specific utilities (macOS only)
copypath                 # Copy current path to clipboard
emoji                    # Emoji insertion
encode64                 # Base64 encode/decode
colored-man-pages        # Colorized man pages
aliases                  # alias-finder helpers
```

### fzf Integration

fzf is configured in `fzf.zsh` with fd as the default file finder:

- **`Ctrl+T`** — fuzzy-find files, with bat syntax-highlighted preview
- **`Alt+C`** — fuzzy-find directories, with eza tree preview
- **`Ctrl+R`** — fuzzy search command history

The `y()` function wraps yazi so the shell changes directory to wherever you navigate inside yazi when you exit it.

### Starship Prompt

The Starship prompt (`~/.config/starship/starship.toml`) shows:

- Current directory (truncated to 5 segments)
- Git branch and status
- Active language/runtime versions (Python, Node.js, Go, Rust, Ruby, Lua, Haskell)
- AWS profile, Docker context
- Background job count
- Command duration (shown when command takes > 500ms)

---

## Docker & Home Assistant (Linux)

On Linux, `run_once_after_60-install-homeassistant-docker.sh.tmpl` automatically installs **Docker Engine** (`docker-ce`) and the **Docker Compose V2 plugin** (`docker compose`), configures firewall rules (`8123/tcp` and `22/tcp`), and deploys **Home Assistant Container** with **HACS** (Home Assistant Community Store) pre-unpacked.

### How It Is Configured

- **Compose file**: `~/home-assistant/compose.yaml` (managed by chezmoi from `home-assistant/compose.yaml`)
- **Config directory**: `~/home-assistant/config/` (mounted to `/config` inside the container)
- **Container name**: `homeassistant`
- **Image**: `ghcr.io/home-assistant/home-assistant:stable`
- **Auto-start policy**: `restart: unless-stopped` — Home Assistant automatically starts on system boot unless you explicitly stop it
- **Web UI**: `http://localhost:8123` (or `http://<linux-host-ip>:8123` on your local network)
- **File ownership handling**: Because the Home Assistant container runs as `root`, files it creates in `~/home-assistant/config/` are owned by `root`. Before every `chezmoi apply`, `run_before_50-chown-homeassistant.sh.tmpl` automatically runs `sudo chown -R "$USER:$USER" ~/home-assistant/config` if any root-owned files are detected.

### Starting, Stopping, and Restarting Home Assistant

You can manage the Home Assistant container using either `docker compose` (recommended) or `docker` directly:

```bash
# --- Using Docker Compose (from anywhere) ---

# Start Home Assistant in the background (creates/recreates container if needed)
docker compose -f ~/home-assistant/compose.yaml up -d

# Stop Home Assistant (keeps container intact, stops running process)
docker compose -f ~/home-assistant/compose.yaml stop

# Start a stopped Home Assistant container
docker compose -f ~/home-assistant/compose.yaml start

# Restart Home Assistant (e.g., after editing configuration.yaml or updating HACS components)
docker compose -f ~/home-assistant/compose.yaml restart

# Stop and remove the container (your data in ~/home-assistant/config is preserved)
docker compose -f ~/home-assistant/compose.yaml down
```

If you `cd ~/home-assistant` first, you can omit `-f ~/home-assistant/compose.yaml`:

```bash
cd ~/home-assistant
docker compose up -d      # Start
docker compose stop       # Stop
docker compose restart    # Restart
docker compose down       # Stop & remove container
```

Or using standard `docker` commands by container name (`homeassistant`):

```bash
docker start homeassistant
docker stop homeassistant
docker restart homeassistant
```

### Checking Status, Logs, and Updating Home Assistant

```bash
# Check if the container is running
docker compose -f ~/home-assistant/compose.yaml ps
# or:
docker ps

# Follow live Home Assistant logs (press Ctrl+C to exit)
docker compose -f ~/home-assistant/compose.yaml logs -f
# or:
docker logs -f homeassistant

# Pull the latest stable Home Assistant image and recreate the container
docker compose -f ~/home-assistant/compose.yaml pull
docker compose -f ~/home-assistant/compose.yaml up -d
```

### Starting, Stopping, and Restarting the Docker Daemon

Docker itself runs as a `systemd` service on Linux and is enabled to start at boot (`systemctl enable --now docker`):

```bash
# Check Docker service status
sudo systemctl status docker

# Start the Docker daemon
sudo systemctl start docker

# Restart the Docker daemon (also restarts Home Assistant unless stopped)
sudo systemctl restart docker

# Stop the Docker daemon completely (stops all running containers)
sudo systemctl stop docker docker.socket
```

### Activating HACS & Connecting a Zigbee Coordinator

**Activating HACS:**
1. HACS is automatically unpacked to `~/home-assistant/config/custom_components/hacs` during bootstrap.
2. Open Home Assistant at `http://localhost:8123` and complete initial onboarding.
3. Go to **Settings** → **Devices & Services** → **Add Integration** → search for **HACS** and follow the GitHub device activation prompt.

**Connecting a Zigbee USB Dongle (ZHA):**
Home Assistant is configured in `compose.yaml` with `privileged: true`, `network_mode: host`, and mounts `/dev:/dev` and `/run/udev:/run/udev:ro`. This allows USB serial dongles (such as Sonoff Zigbee 3.0 USB Dongle Plus or Home Assistant SkyConnect) to be hotplugged without Docker failing to start if the dongle is unplugged.
1. Plug your USB dongle into the Linux host (using a USB 2.0 extension cable is recommended to avoid 2.4 GHz USB 3.0 interference).
2. The bootstrap script automatically adds `$USER` to the `dialout` and `tty` groups and disables `ModemManager` so it cannot lock serial ports.
3. In Home Assistant (`http://<host-ip>:8123`), go to **Settings** → **Devices & Services** → **Add Integration** → **Zigbee Home Automation (ZHA)**.
4. Select the detected serial port (e.g., `/dev/serial/by-id/usb-ITead_Sonoff_Zigbee_3.0_USB_Dongle_Plus...`) and follow the setup wizard.

---

## Installed Tools

### macOS — Homebrew Packages

**Shell & prompt**
- `zsh`, `starship`, `thefuck`, `zoxide`, `direnv`

**Shell history**
- `fzf`, `navi`

**File tools**
- `eza`, `bat`, `bat-extras`, `yazi`, `fd`, `ripgrep`, `tree`, `trash`, `duf`

**Dev tools**
- `neovim`, `git`, `git-filter-repo`, `diff-so-fancy`, `lazygit`, `gh`, `just`, `fnm`, `jq`, `glow`, `curl`, `wget`, `rsync`

**Security & secrets**
- `age`, `sops`, `gnupg`, `detect-secrets`

**Infra / cloud**
- `ansible`, `opentofu`, `kubernetes-cli`, `tailscale`

**Networking & diagnostics**
- `mtr`, `nmap`, `iperf3`, `speedtest-cli`, `curlie`

**Misc CLI**
- `fastfetch`, `btop`, `tmux`, `mas`, `opencode`, `chezmoi`

### macOS — Homebrew Casks (GUI Apps)

| App | Purpose |
|-----|---------|
| Ghostty | Terminal emulator |
| OrbStack | Docker / Linux VMs |
| 1Password + CLI | Password manager + secrets |
| Raycast | Spotlight replacement |
| LinearMouse | Mouse customization |
| AltTab | Window switcher |
| Stats | System stats in menu bar |
| Thaw | Unfreeze / utility app |
| Figma | Design tool |
| draw.io | Diagramming |
| Postman | API client |
| Cyberduck | FTP/S3/cloud storage client |
| LocalSend | Local file transfer |
| Keka | Archive manager |
| Discord, WhatsApp | Messaging |
| Wireshark | Network analyzer |
| BasicTeX | Minimal TeX distribution |
| cmux | Terminal multiplexer UI |
| Symbols Only Nerd Font | Nerd Font icons for terminal |

### Linux — Native Packages

**Arch** (via `pacman` + install scripts):
- `pacman`: `zsh`, `starship`, `eza`, `bat`, `fzf`, `fd`, `ripgrep`, `zoxide`, `direnv`, `git`, `neovim`, `lazygit`, `github-cli`, `jq`, `tmux`, `age`, `gnupg`, `curl`, `wget`, `rsync`, `tree`, `btop`, `fastfetch`, `chezmoi`, `just`, `sops`, `tlp`, `ghostty`, `keyd`, `docker`, `docker-compose`
- Install scripts: `fnm`, `oh-my-zsh`, `zsh-autosuggestions`, `zsh-syntax-highlighting`

**Debian/Ubuntu** (via `apt`, official repos/PPAs, and install scripts):
- `apt`: `zsh`, `eza`, `bat`, `fd-find`, `ripgrep`, `direnv`, `git`, `neovim`, `jq`, `tmux`, `age`, `gnupg`, `curl`, `wget`, `rsync`, `tree`, `btop`, `python3`, `python3-pip`, `python3-selenium`, `python3-venv`, `fail2ban`, `gnome-tweaks`, `silversearcher-ag`, `tlp`, `keyd`, `xz-utils`, `make`, `unzip`, `openssh-server`, `chromium-chromedriver`, `software-properties-common`
- Official APT repos & PPAs: `gh` (GitHub CLI APT repo), `firefox` (Mozilla APT repo — native `.deb` replacing Snap), `google-chrome-stable` (Google APT repo), `docker-ce` & `docker-compose-plugin` (Docker APT repo), `fastfetch` (`ppa:zhangsongcui3371/fastfetch`), `ghostty` (`ppa:mkick/ghostty`)
- Install scripts / GitHub releases: `fzf` (GitHub releases — `apt` version is too old for `fzf --zsh`), `starship`, `zoxide`, `lazygit`, `chezmoi`, `just`, `sops`, `fnm`, `oh-my-zsh`, `zsh-autosuggestions`, `zsh-syntax-highlighting`

---

## Adding New Files

### Track a New Dotfile

```bash
# Add a file to chezmoi source
chezmoi add ~/.config/some/config-file

# Add as encrypted
chezmoi add --encrypt ~/.config/some/secret-file
```

### Track a Directory

```bash
chezmoi add ~/.config/some-tool/
```

### Exclude a File/Directory from Tracking

Add a pattern to `.chezmoiignore`:

```bash
# In ~/.local/share/chezmoi/.chezmoiignore
dot_config/some-tool/
```

---

## Troubleshooting

### age decryption fails on apply

**Error:** `chezmoi: decrypt: ...`

**Cause:** The age key at `~/.config/chezmoi/key.txt` is wrong or corrupted (note: if `key.txt` is completely absent, `.chezmoiignore` skips encrypted files automatically).

**Solution:**
1. Retrieve the age private key from 1Password
2. Write it to `~/.config/chezmoi/key.txt`:
   ```bash
   mkdir -p ~/.config/chezmoi
   nano ~/.config/chezmoi/key.txt
   chmod 600 ~/.config/chezmoi/key.txt
   ```
3. Re-run: `chezmoi apply`

### Prompts Appear on Every `chezmoi apply`

**Cause:** `~/.config/chezmoi/chezmoi.toml` is missing or incomplete.

**Solution:** Answer all prompts once — chezmoi saves them to `chezmoi.toml`. If the file was deleted, re-run `chezmoi init --apply larham/linux-dotfiles` or create it manually:

```toml
# ~/.config/chezmoi/chezmoi.toml
encryption = "age"

[data]
  email = "you@example.com"
  name = "Your Name"
  gpgKeyId = ""            # leave empty to disable GPG signing
  hasOrbStack = true       # set to false on Linux (ignored by templates anyway)

[age]
  identity = "~/.config/chezmoi/key.txt"
  recipient = "age16mv7ycglyrn68pnrfv6r2pf72swjffn20sdehfdha7l5sqlr3s2qa2mlsl"
```

### Docker Permission Denied on Linux

**Error:** `permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock`

**Cause:** Your user was added to the `docker` group during bootstrap, but the current login session has not picked up the new group membership yet.

**Solution:** Log out and log back in, or activate the group in your current shell:

```bash
newgrp docker
```

### Home Assistant Container Not Starting or Port 8123 Unreachable

**Solution:**
1. Check whether the Docker daemon and `homeassistant` container are running:
   ```bash
   sudo systemctl status docker
   docker compose -f ~/home-assistant/compose.yaml ps -a
   ```
2. Inspect the container logs for configuration errors:
   ```bash
   docker compose -f ~/home-assistant/compose.yaml logs --tail=100
   ```
3. Restart the container after fixing any issues in `~/home-assistant/config/configuration.yaml`:
   ```bash
   docker compose -f ~/home-assistant/compose.yaml restart
   ```

### Packages Not Installing (macOS)

**Cause:** `run_once_before_20` script already ran and chezmoi won't re-run it.

**Solution:** Run `brew bundle` manually:

```bash
brew bundle --file="$HOME/.local/share/chezmoi/Brewfile"
```

### Packages Not Installing (Linux)

**Cause:** Same — `run_once_` script already ran.

**Solution:** Reset the script state and re-apply, or run the install commands manually:

```bash
chezmoi state delete-bucket --bucket=scriptState
chezmoi apply
```

### `chezmoi diff` Shows Unexpected Changes

chezmoi detected a difference between source and target. Preview with:

```bash
chezmoi diff
```

To overwrite target with source (apply source → target):
```bash
chezmoi apply
```

To overwrite source with target (update source from your edits):
```bash
chezmoi re-add ~/.config/some/file
```

### zsh-syntax-highlighting Not Working

The plugin is cloned automatically by the bootstrap script. If it's missing, clone it manually:

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting \
  ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting
```

Then reload: `source ~/.zshrc`

### GPG Commit Signing Not Working

**Symptoms:** `gpg: signing failed: Inappropriate ioctl for device` or commits are not signed.

**Solution:**

1. Verify the key ID in your chezmoi config matches an existing key:
   ```bash
   gpg --list-secret-keys --keyid-format=long
   git config --global user.signingKey
   ```

2. Ensure `GPG_TTY` is set (already in `env.zsh`):
   ```bash
   export GPG_TTY=$(tty)
   ```

3. If you want to disable signing, re-init with an empty GPG key ID or edit `~/.config/chezmoi/chezmoi.toml` and set `gpgKeyId = ""`, then run `chezmoi apply`.

### Default Shell Not Set to Zsh (Linux)

The bootstrap script runs `chsh -s $(which zsh)` automatically. If it didn't take effect:

```bash
# Verify zsh path
which zsh

# Set manually
chsh -s /usr/bin/zsh

# Log out and back in for the change to apply
```

### Ghostty Two-Finger Scroll Shows Command History Instead of Scrolling Output

**Symptoms:** Using a two-finger touchpad scroll up/down at the shell prompt cycles through previous Zsh commands (as if pressing the `Up`/`Down` arrow keys) instead of scrolling the terminal's output buffer.

**Cause:** A full-screen/TUI program (`btop`, `lazygit`, `yazi`, `nvim`, `less`, `opencode`, or an interrupted `ssh` session) exited without restoring the primary terminal screen, leaving Ghostty stuck in the **alternate screen buffer** (`?1049h`) with **Alternate Scroll Mode** (`?1007h`) active. In that mode, Ghostty translates touchpad scroll events into `Up`/`Down` arrow key presses.

**How to manually fix it:**

Run any one of the following commands in the affected terminal tab:

```bash
# Option 1: Exit the alternate screen without clearing your terminal (recommended)
tput rmcup

# Option 2: Explicitly exit alternate screen (?1049l) and disable alternate scroll (?1007l)
printf '\e[?1049l\e[?1007l'

# Option 3: Full terminal reset (also clears the screen)
reset
```

> **Tip:** If you are actively inside a full-screen TUI app (like `less` or `btop`) and want to scroll Ghostty's outer scrollback buffer without exiting the app, hold **`Shift`** while two-finger scrolling.

---

## License

Personal dotfiles — feel free to use anything here as inspiration.
