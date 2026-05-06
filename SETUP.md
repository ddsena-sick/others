# Proxmox VM — Claude Code Session Setup

---

## Schritt 1: Für den Host-Benutzer (ddsena-sick)

Diese Schritte führst **du** einmalig aus, bevor die Claude-Session auf der VM startet.

### 1.1 GitHub Personal Access Token erstellen

1. Gehe zu: https://github.com/settings/tokens
2. "Generate new token (classic)"
3. Scopes: `repo`, `read:org`, `gist`
4. Token kopieren und sicher aufbewahren — du gibst ihn der VM beim Setup

### 1.2 Anthropic API Key bereitstellen

- API Key aus https://console.anthropic.com/settings/keys holen (oder neuen erstellen)
- Den Key bereithalten für Schritt 2.4 auf der VM

### 1.3 VM in Proxmox erstellen

- OS: Ubuntu 24.04 LTS (oder Debian 12)
- RAM: mindestens 2 GB
- Disk: mindestens 20 GB
- Netzwerk: eigenes isoliertes VLAN oder Bridge nach Bedarf

---

## Schritt 2: Für die Claude-Session auf der VM

Diese Schritte führt die **neue Claude-Session** in der Proxmox-VM aus.

### 2.1 System-Dependencies installieren

```bash
sudo apt update && sudo apt install -y \
  git curl wget unzip build-essential \
  python3 python3-pip nodejs npm
```

### 2.2 GitHub CLI installieren

```bash
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
  | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] \
  https://cli.github.com/packages stable main" \
  | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update && sudo apt install -y gh
```

### 2.3 Git und GitHub konfigurieren

```bash
git config --global user.name "ddsena-sick"
git config --global user.email "ddsena@gmx.de"
```

Mit dem PAT aus Schritt 1.1 einloggen:

```bash
gh auth login
# Wähle: GitHub.com → HTTPS → Paste an authentication token
# Token eingeben
```

### 2.4 Claude Code installieren

```bash
npm install -g @anthropic-ai/claude-code
```

API Key setzen:

```bash
export ANTHROPIC_API_KEY="sk-ant-..."   # deinen Key hier eintragen
# Dauerhaft speichern:
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.bashrc
source ~/.bashrc
```

### 2.5 Dieses Repo klonen

```bash
git clone https://github.com/ddsena-sick/others.git ~/others
cd ~/others
```

### 2.6 Arbeitsverzeichnis anlegen und Claude Code starten

```bash
mkdir -p ~/projects
cd ~/projects
claude
```

---

## Hinweise

- Diese VM ist **isoliert** vom Haupt-Setup (kein Zugriff auf ESP-IDF, Hubelino-Projekt, etc.)
- Repos werden separat geklont — kein geteilter State mit der Haupt-Session
- Für projektspezifische Repos: `gh repo clone ddsena-sick/<reponame>`
