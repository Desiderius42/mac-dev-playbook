# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fork of [geerlingguy/mac-dev-playbook](https://github.com/geerlingguy/mac-dev-playbook) — an Ansible playbook for automated macOS developer machine provisioning. Extended with an Ubuntu VM playbook (`UbuntuVM.yml`) for a Freebox home server (SABnzbd, Plex).

## Key Commands

```bash
# Install Ansible dependencies (roles + collections)
ansible-galaxy install -r requirements.yml

# Run full macOS playbook (prompts for sudo password)
ansible-playbook main.yml -i inventory -K

# Run specific tags only
ansible-playbook main.yml -K --tags "homebrew"
ansible-playbook main.yml -K --tags "mas"
ansible-playbook main.yml -K --tags "dotfiles,homebrew"

# Run Ubuntu VM playbook (Freebox server)
ansible-playbook UbuntuVM.yml --ask-vault-pass -v

# Validate syntax
ansible-playbook main.yml --syntax-check

# Dry run (no changes)
ansible-playbook main.yml --check

# Lint
yamllint .
```

## Architecture

**Two playbooks:**
- `main.yml` — macOS provisioning (localhost). Uses `geerlingguy.mac` collection for Homebrew, Mac App Store, Dock management.
- `UbuntuVM.yml` — Ubuntu/Freebox server provisioning (remote SSH to `192.168.1.157`). Installs SABnzbd, Plex, cron jobs.

**Configuration override pattern:** `default.config.yml` holds all variables. Users create a `config.yml` (git-ignored) to override defaults. Command-line vars take highest priority.

**Tag-based modularity:** Tasks are selectively runnable via tags: `homebrew`, `mas`, `dotfiles`, `terminal`, `osx`, `extra-packages`, `sublime-text`, `post`.

**External dependencies** (installed via `requirements.yml`):
- Role `elliotweiser.osx-command-line-tools` — Xcode CLI tools
- Role `geerlingguy.dotfiles` — dotfiles management
- Collection `geerlingguy.mac` — Homebrew, MAS, Dock, macOS modules

## Key Files

- `default.config.yml` — All configurable variables (packages, casks, MAS apps, feature flags)
- `inventory` — Defines localhost (macOS) and UbuntuVM (Freebox) hosts
- `tasks/` — Custom task modules (osx settings, terminal, sublime, extra-packages, sudoers)
- `files/` — Static configs (SABnzbd, Sublime Text themes, Terminal themes)
- `tests/` — CI test config with minimal package subset

## CI

GitHub Actions (`.github/workflows/ci.yml`) runs on PRs, pushes to master, and weekly. Tests syntax check + Homebrew tag execution + idempotence verification on macOS.
