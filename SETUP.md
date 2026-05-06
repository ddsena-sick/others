# Proxmox VM — Isolated Claude Code Session Setup

Dieses System ist vollständig unabhängig. Es gibt keinen gemeinsamen State,
keine geteilten Repos, keine Verbindung zu bestehenden Accounts oder Projekten.

---

## Schritt 1: Für den Host-Benutzer (einmalig, vor VM-Start)

### 1.1 Neuen GitHub Account erstellen

- Gehe zu: https://github.com/join
- Neuen Account mit separater E-Mail-Adresse anlegen
- Merke dir: `<neuer-github-username>` und `<neue-email>`

### 1.2 Personal Access Token für den neuen Account erstellen

1. Im neuen Account: https://github.com/settings/tokens
2. "Generate new token (classic)"
3. Scopes: `repo`, `read:org`, `gist`
4. Token kopieren — wird in Schritt 2.3 auf der VM gebraucht

### 1.3 Anthropic API Key bereitstellen

- Neuen Key erstellen unter: https://console.anthropic.com/settings/keys
- Separat vom bestehenden Key halten

### 1.4 VM in Proxmox erstellen

- OS: Ubuntu 24.04 LTS oder Debian 12
- RAM: mindestens 2 GB
- Disk: mindestens 20 GB
- Netzwerk: isolierte Bridge oder eigenes VLAN — kein Zugriff auf andere VMs

---

## Schritt 2: Für die Claude-Session auf der VM

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
git config --global user.name "<neuer-github-username>"
git config --global user.email "<neue-email>"
```

Mit dem PAT aus Schritt 1.2 einloggen:

```bash
gh auth login
# Wähle: GitHub.com → HTTPS → Paste an authentication token
# Token eingeben
```

### 2.4 Claude Code installieren

```bash
npm install -g @anthropic-ai/claude-code
```

API Key setzen (den aus Schritt 1.3):

```bash
echo 'export ANTHROPIC_API_KEY="sk-ant-..."' >> ~/.bashrc
source ~/.bashrc
```

### 2.5 Arbeitsverzeichnis anlegen und Claude Code starten

```bash
mkdir -p ~/projects
cd ~/projects
claude
```

---

## Wichtige Hinweise

- Kein Zugriff auf andere GitHub Accounts oder bestehende Repos
- Kein geteilter API Key mit anderen Instanzen
- Alle Repos werden frisch im neuen Account angelegt
- Diese VM ist das einzige System, das den neuen Account nutzt
