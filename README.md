# Mac Dev Playbook

Fork de [geerlingguy/mac-dev-playbook](https://github.com/geerlingguy/mac-dev-playbook). Deux playbooks :

- **`main.yml`** — provisioning macOS généraliste (upstream)
- **`dev-workstation.yml`** — station de dev pure (React Native + Rust + Claude Code)
- **`UbuntuVM.yml`** — serveur home Freebox (Plex, SABnzbd)

## Installation dev-workstation (Mac neuf)

### 1. Ouvrir Terminal

Spotlight (`Cmd+Espace`) > "Terminal"

### 2. Installer Homebrew (installe aussi Xcode Command Line Tools automatiquement, ~10 min)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Suivre les instructions affichées à la fin :

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zshrc
eval "$(/opt/homebrew/bin/brew shellenv)"
```

### 3. Installer 1Password (mots de passe nécessaires pour la suite)

```bash
brew install --cask 1password
```

Ouvrir 1Password, se connecter, récupérer ses mots de passe (dont Apple ID).

Activer l'agent SSH : 1Password > Settings > Developer > "Set up SSH Agent". Cela permet à Git d'utiliser les clés SSH stockées dans 1Password.

Vérifier que la connexion SSH fonctionne :

```bash
ssh -T git@github.com
```

### 4. Se connecter au Mac App Store

Ouvrir App Store, se connecter avec son Apple ID (nécessaire pour installer Xcode via le playbook).

### 5. Installer Ansible et Claude Code

```bash
brew install ansible node
npm install -g @anthropic-ai/claude-code
```

Claude Code est disponible dès maintenant pour aider à debugger si le playbook échoue.

### 6. Installer les casks nécessitant sudo interactif (incompatibles avec le playbook)

```bash
brew install --cask google-drive tailscale
```

### 7. Cloner et lancer le playbook

```bash
git clone git@github.com:Desiderius42/mac-dev-playbook.git
cd mac-dev-playbook
ansible-galaxy install -r requirements.yml
ansible-playbook dev-workstation.yml -i inventory -K
```

`-K` demande le mot de passe sudo (mot de passe de session Mac).

Le playbook installe : brew packages, cask apps (VS Code, Docker, Android Studio...), Xcode (MAS), CocoaPods (gem), Claude Code (npm), Rust + cross-compilation targets, config Claude Code (~/.claude/).

### 8. Post-install manuels

**Rust dans le PATH :**

```bash
source $HOME/.cargo/env
```

Pour rendre permanent, ajouter dans `~/.zshrc` :

```bash
source $HOME/.cargo/env
```

**Java — lier openjdk@17 :**

```bash
sudo ln -sfn $(brew --prefix openjdk@17)/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-17.jdk
```

**Android Studio** — au premier lancement :

1. Installer Android SDK API 35
2. SDK Tools > installer **NDK (Side by side)** v27+
3. Ajouter dans `~/.zshrc` :

```bash
export ANDROID_HOME=$HOME/Library/Android/sdk
export ANDROID_NDK_HOME=$ANDROID_HOME/ndk/<version>
export PATH=$ANDROID_HOME/emulator:$ANDROID_HOME/platform-tools:$PATH
```

**Claude Code plugins :**

```bash
claude plugins install ouroboros swift-lsp rust-analyzer-lsp
```

### 9. Vérification

```bash
node --version      # >= 20
rustc --version     # stable
pod --version       # >= 1.15
java --version      # 17
claude --version
gh auth login       # authentification GitHub
```

## Lancer des tags spécifiques

```bash
ansible-playbook dev-workstation.yml -K --tags "homebrew"
ansible-playbook dev-workstation.yml -K --tags "rust"
ansible-playbook dev-workstation.yml -K --tags "claude-code"
```

Tags disponibles : `homebrew`, `mas`, `extra-packages`, `rust`, `claude-code`

## Réinstaller des apps supprimées

Les apps non-dev sont commentées dans `default.config.yml`. Décommenter la ligne et relancer :

```bash
ansible-playbook dev-workstation.yml -i inventory -K
```

## Overriding Defaults

Créer un fichier `config.yml` (git-ignored) pour surcharger `default.config.yml` sans le modifier :

```yaml
homebrew_installed_packages:
  - git
  - node

homebrew_cask_apps:
  - docker
  - firefox
```

## Installation Freebox (Ubuntu VM)

```bash
# Modifier l'IP dans le fichier inventory
ansible-playbook UbuntuVM.yml --ask-vault-pass -v
```

Post-install :
- Configurer Plex : `http://<IP>:32400/web`
- Réinstaller les mots de passe Newsgroup
- Freebox reboot : https://github.com/PabloLec/freebox_reboot

## Origine

Fork de [Jeff Geerling](https://www.jeffgeerling.com/) — [geerlingguy/mac-dev-playbook](https://github.com/geerlingguy/mac-dev-playbook)
