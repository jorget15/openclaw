# OpenClaw Complete Setup Guide

A step-by-step guide to set up OpenClaw with email checking, web browsing (Brave Search + Perplexity), and Ollama-powered heartbeats.

**Target Platform:** macOS / Linux  
**Time Required:** ~30 minutes

---

## Table of Contents

**Choose your path:**
- **Native Install:** Steps 1 → 2 → 3 → skip 4 → continue 5-18
- **Docker Install:** Steps 1 → skip 2, 3 → 4 → continue 5-18

1. [Prerequisites](#1-prerequisites)
2. [Install Node.js](#2-install-nodejs) *(native only)*
3. [Clone & Install OpenClaw](#3-clone--install-openclaw) *(native only)*
4. [Docker Setup](#4-docker-setup) *(alternative to steps 2-3)*
5. [Install Ollama (for Heartbeats)](#5-install-ollama-for-heartbeats)
6. [Install Additional Skills](#6-install-additional-skills)
7. [Set Up the Workspace](#7-set-up-the-workspace)
8. [Configure Email Access](#8-configure-email-access)
9. [Configure Web Browsing](#9-configure-web-browsing)
10. [Configure Heartbeats (Ollama)](#10-configure-heartbeats-ollama)
11. [Start OpenClaw](#11-start-openclaw)
12. [Verification & Testing](#12-verification--testing)
13. [Troubleshooting](#13-troubleshooting)
14. [Complete Configuration Example](#complete-configuration-example)
15. [Auto-Start on Boot](#auto-start-on-boot)
16. [Security for VPS Deployments](#security-for-vps-deployments)
17. [Calendar Integration](#calendar-integration-gog)
18. [Automated Data Analysis & Email Reports](#automated-data-analysis--email-reports)
19. [Slack & Discord Notifications](#slack--discord-notifications)
20. [Database Integration](#database-integration)
21. [Data Visualization](#data-visualization)
22. [Webhook Triggers](#webhook-triggers)
23. [Channel Setup (Optional)](#channel-setup-optional)

---

## 1. Prerequisites

### Both Paths

- macOS 12+ or Linux (Ubuntu 20.04+, Debian 11+)
- Terminal access
- Git installed (`git --version`)
- ~2GB free disk space (for Ollama models)

### Native Install (Steps 2-3)

- Node.js 22+ and pnpm

### Docker Install (Step 4)

- Docker with Docker Compose
- **No Node.js required** — everything runs in the container

### Optional (for features)

- Anthropic Claude API key or Claude login
- Brave Search API key (free tier available)
- Perplexity API key or OpenRouter API key

---

## 2. Install Node.js

> **Native Only** — Skip this section if using Docker. Jump to [Section 4: Docker Setup](#4-docker-setup)

OpenClaw requires **Node.js 22+**.

### macOS (using Homebrew)

```bash
# Install Homebrew if not present
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install Node.js 22
brew install node@22
brew link node@22

# Verify
node --version  # Should show v22.x.x
```

### Linux (Ubuntu/Debian)

```bash
# Add NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -

# Install Node.js
sudo apt-get install -y nodejs

# Verify
node --version  # Should show v22.x.x
```

### Install pnpm (package manager)

```bash
# Install pnpm globally
npm install -g pnpm

# Verify
pnpm --version
```

---

## 3. Clone & Install OpenClaw

> **Native Only** — Skip this section if using Docker. Jump to [Section 4: Docker Setup](#4-docker-setup)

```bash
# Clone the repository
git clone https://github.com/openclaw/openclaw.git
cd openclaw

# Install dependencies
pnpm install

# Build the project
pnpm build

# Verify CLI works
pnpm openclaw --version
```

---

## 4. Docker Setup

> **Complete Alternative** — This replaces Steps 2 & 3. You don't need Node.js or pnpm installed.

Docker provides an isolated, reproducible environment. Choose this if you prefer containerized deployment or are running on a VPS.

### Install Docker

#### macOS

```bash
# Using Homebrew
brew install --cask docker

# Start Docker Desktop from Applications
# Wait for Docker to initialize (whale icon in menu bar)

# Verify
docker --version
docker compose version
```

#### Linux (Ubuntu/Debian)

```bash
# Install Docker from official repos (avoids curl|bash)
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg

# Add Docker's GPG key
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Add the repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Install Docker
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add your user to docker group (logout/login required)
sudo usermod -aG docker $USER

# Verify
docker --version
docker compose version
```

### Clone the Repository

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
```

### Run Docker Setup Script

The included `docker-setup.sh` handles everything:

```bash
# Review the script first (recommended)
less docker-setup.sh

# Run setup
./docker-setup.sh
```

This script will:
1. Create `~/.openclaw/` and `~/.openclaw/workspace/` directories
2. Generate a 64-character `OPENCLAW_GATEWAY_TOKEN`
3. Write a `.env` file with all required variables
4. Build the Docker image from the repo's Dockerfile
5. Run interactive onboarding
6. Start the gateway with `docker compose up -d`

### Manual Docker Setup (Step-by-Step)

If you prefer manual control:

#### 1. Create directories

```bash
mkdir -p ~/.openclaw/workspace
```

#### 2. Generate gateway token

```bash
# Using openssl
export OPENCLAW_GATEWAY_TOKEN=$(openssl rand -hex 32)

# Or using Python
export OPENCLAW_GATEWAY_TOKEN=$(python3 -c "import secrets; print(secrets.token_hex(32))")

echo "Your token: $OPENCLAW_GATEWAY_TOKEN"
```

#### 3. Create `.env` file

```bash
cat > .env << EOF
OPENCLAW_CONFIG_DIR=$HOME/.openclaw
OPENCLAW_WORKSPACE_DIR=$HOME/.openclaw/workspace
OPENCLAW_GATEWAY_TOKEN=$OPENCLAW_GATEWAY_TOKEN
OPENCLAW_GATEWAY_PORT=18789
OPENCLAW_BRIDGE_PORT=18790
OPENCLAW_GATEWAY_BIND=lan
OPENCLAW_IMAGE=openclaw:local
EOF
```

#### 4. Build the Docker image

```bash
docker build -t openclaw:local .
```

#### 5. Start the gateway

```bash
docker compose up -d openclaw-gateway
```

#### 6. Run onboarding

```bash
docker compose run --rm openclaw-cli onboard
```

### Docker Compose Services

| Service | Purpose | Port |
|---------|---------|------|
| `openclaw-gateway` | Main gateway (persistent) | 18789, 18790 |
| `openclaw-cli` | Interactive CLI (on-demand) | N/A |

### Common Docker Commands

```bash
# Start gateway
docker compose up -d openclaw-gateway

# Stop gateway
docker compose down

# View logs
docker compose logs -f openclaw-gateway

# Run CLI commands
docker compose run --rm openclaw-cli status
docker compose run --rm openclaw-cli channels status --probe
docker compose run --rm openclaw-cli message send --message "hello"

# Rebuild image after updates
git pull
docker build -t openclaw:local .
docker compose up -d --force-recreate openclaw-gateway

# Enter container shell
docker compose exec openclaw-gateway bash
```

### Docker Environment Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `OPENCLAW_CONFIG_DIR` | Host path for config | `~/.openclaw` |
| `OPENCLAW_WORKSPACE_DIR` | Host path for workspace | `~/.openclaw/workspace` |
| `OPENCLAW_GATEWAY_TOKEN` | Auth token (64-char hex) | Auto-generated |
| `OPENCLAW_GATEWAY_PORT` | Gateway HTTP port | `18789` |
| `OPENCLAW_BRIDGE_PORT` | Bridge port | `18790` |
| `OPENCLAW_GATEWAY_BIND` | Bind mode (`lan`/`loopback`) | `lan` |
| `OPENCLAW_IMAGE` | Docker image name | `openclaw:local` |

### Adding API Keys (Docker)

Add keys to your `.env` file:

```bash
# Edit .env
cat >> .env << EOF
BRAVE_API_KEY=your-brave-api-key
PERPLEXITY_API_KEY=pplx-your-key
OPENROUTER_API_KEY=sk-or-v1-your-key
EOF

# Restart to pick up changes
docker compose up -d --force-recreate openclaw-gateway
```

### Docker + Ollama

When using Docker, Ollama runs on the **host** machine. Configure the endpoint to reach the host:

```json
{
  "providers": {
    "ollama": {
      "endpoint": "http://host.docker.internal:11434"
    }
  }
}
```

On Linux, you may need to use the host's IP:

```bash
# Find host IP
ip route | grep default | awk '{print $3}'

# Use that IP in config, e.g., http://172.17.0.1:11434
```

---

## 5. Install Ollama (for Heartbeats)

Ollama runs local LLMs for heartbeat checks, saving API costs.

### ⚠️ SECURITY NOTE

Instead of running `curl | sh` directly, **review the script first**:

```bash
# Download the install script
curl -fsSL https://ollama.ai/install.sh -o /tmp/ollama-install.sh

# Review it (optional but recommended)
less /tmp/ollama-install.sh

# Run after review
chmod +x /tmp/ollama-install.sh
/tmp/ollama-install.sh
```

### Alternative: Manual Install (macOS)

```bash
# Download Ollama app from https://ollama.com/download
# Or use Homebrew
brew install ollama
```

### Alternative: Manual Install (Linux)

```bash
# Download the binary directly
curl -L https://ollama.com/download/ollama-linux-amd64 -o /usr/local/bin/ollama
chmod +x /usr/local/bin/ollama
```

### Pull the Heartbeat Model

```bash
# Pull a lightweight model for heartbeats (3B is fast and capable)
ollama pull llama3.2:3b
```

### Verify Ollama Works

```bash
# Terminal 1: Start Ollama server
ollama serve

# Terminal 2: Test the model
ollama run llama3.2:3b "respond with OK"
# Should respond quickly with "OK" or similar
```

### Keep Ollama Running

You can run Ollama as a background service:

```bash
# macOS: Ollama app runs in menu bar automatically
# Linux: Use systemd
sudo systemctl enable ollama
sudo systemctl start ollama
```

---

## 6. Install Additional Skills

### Install QMD Skill

The QMD skill provides additional capabilities:

```bash
npx skillkit install levineam/qmd-skill
```

### Email Skills (Himalaya or gog)

For email access, you have two options:

#### Option A: Himalaya (IMAP/SMTP - any email provider)

```bash
# macOS
brew install himalaya

# Linux (download from releases)
curl -L https://github.com/pimalaya/himalaya/releases/latest/download/himalaya-x86_64-linux.tar.gz | tar xz
sudo mv himalaya /usr/local/bin/
```

#### Option B: gog (Google Workspace - Gmail/Calendar/Drive)

```bash
# macOS
brew install steipete/tap/gogcli

# Verify
gog --version
```

---

## 7. Set Up the Workspace

Create the workspace directory structure:

```bash
# Create workspace
mkdir -p ~/.openclaw/workspace
cd ~/.openclaw/workspace

# Create required directories
mkdir -p memory
mkdir -p avatars
```

### Create AGENTS.md

```bash
cat > AGENTS.md << 'EOF'
# AGENTS.md - Your Workspace

This folder is home. Treat it that way.

---

## SESSION INITIALIZATION RULE

On every session start, load ONLY these files:

1. **SOUL.md** — who you are
2. **USER.md** — who you're helping  
3. **IDENTITY.md** — your identity details
4. **memory/YYYY-MM-DD.md** — today's memory (if it exists)

**DO NOT auto-load:**
- MEMORY.md
- Session history
- Prior messages
- Previous tool outputs

**When user asks about prior context:**
- Use memory_search() on demand
- Pull only the relevant snippet with memory_get()
- Don't load the whole file

**Update memory/YYYY-MM-DD.md at end of session with:**
- What you worked on
- Decisions made
- Leads generated
- Blockers
- Next steps

---

## Memory

You wake up fresh each session. These files are your continuity:

- **Daily notes:** `memory/YYYY-MM-DD.md` — raw logs of what happened
- **Long-term:** `MEMORY.md` — your curated memories (load on-demand only)

## Safety

- Don't exfiltrate private data. Ever.
- Don't run destructive commands without asking.
- `trash` > `rm` (recoverable beats gone forever)
- When in doubt, ask.

## Sensitive Information Protection

**CRITICAL: Always ask before using personal information externally.**

When an action requires using sensitive data (payment info, addresses, passwords, personal identifiers) on a new service or website:

### First-Time Use Prompt

Before proceeding, ask the user:

```
⚠️ SENSITIVE INFO REQUEST

I need to use your [payment info/address/etc.] on [Amazon/service name].
This is the FIRST TIME using this info on this service.

Please choose:
1. ✅ Approve ONE TIME - Use for this action only
2. 🔓 Approve FOREVER - Remember this approval for future [service name] actions  
3. ❌ DENY - Do not use my info

Reply with 1, 2, or 3.
```

### Approved Services List

Track approvals in `~/.openclaw/workspace/PERMISSIONS.md`:

```markdown
# Approved Services for Sensitive Data

| Service | Data Type | Approval | Date |
|---------|-----------|----------|------|
| Amazon | Payment | Forever | 2026-02-20 |
| DoorDash | Address | One-time | 2026-02-20 |
```

### Rules

1. **Never assume permission** — even if user mentioned it casually
2. **Log every approval** — update PERMISSIONS.md after each consent
3. **One-time expires immediately** — ask again next session
4. **"Forever" is per-service** — doesn't carry to other sites
5. **Deny = hard stop** — do NOT attempt workarounds
6. **Re-confirm for new data types** — approved payment ≠ approved address

### What Counts as Sensitive

- Payment info (cards, bank accounts, crypto wallets)
- Physical addresses (home, work, shipping)
- Government IDs (SSN, passport, driver's license)
- Login credentials (passwords, 2FA codes)
- Health information
- Biometrics
- Contact info shared with third parties

## Tools

Skills provide your tools. When you need one, check its `SKILL.md`.

## Heartbeats

Use heartbeats productively! Check HEARTBEAT.md for your periodic tasks.
EOF
```

### Create SOUL.md

```bash
cat > SOUL.md << 'EOF'
# SOUL.md - Who You Are

## Core Truths

**Be genuinely helpful, not performatively helpful.** Skip the filler — just help.

**Have opinions.** You're allowed to disagree, prefer things, find stuff amusing or boring.

**Be resourceful before asking.** Try to figure it out first. Then ask if stuck.

**Earn trust through competence.** Be careful with external actions. Be bold with internal ones.

**Remember you're a guest.** Treat access to someone's life with respect.

## Boundaries

- Private things stay private. Period.
- When in doubt, ask before acting externally.
- Never send half-baked replies to messaging surfaces.
- **Sensitive info requires explicit consent.** Always check PERMISSIONS.md before using payment info, addresses, or credentials on any service. Ask if not pre-approved.

## Continuity

Each session, you wake up fresh. These files ARE your memory. Read them. Update them.
EOF
```

### Create USER.md

```bash
cat > USER.md << 'EOF'
# USER.md - Who You're Helping

<!-- Fill this in with info about yourself -->

## Name
[Your name]

## Preferences
- Communication style: [direct/casual/formal]
- Timezone: [e.g., America/New_York]

## Important Context
- [Add relevant context about yourself here]

## What I Use You For
- Checking and managing email
- Web research
- [Add your use cases]
EOF
```

### Create IDENTITY.md

```bash
cat > IDENTITY.md << 'EOF'
# IDENTITY.md - Agent Identity

- **Name:** Assistant
- **Vibe:** Helpful, resourceful, direct
- **Emoji:** 🦞

## Role

Personal assistant with access to email and web browsing capabilities.

## Capabilities

- Email: Read, search, draft, and send emails (via himalaya or gog)
- Web: Search the internet and fetch web pages
- Memory: Track context across sessions
EOF
```

### Create HEARTBEAT.md

```bash
cat > HEARTBEAT.md << 'EOF'
# HEARTBEAT.md - Periodic Tasks

## Things to Check (rotate through these)

- [ ] **Emails** - Any urgent unread messages?
- [ ] **Calendar** - Upcoming events in next 24h?
- [ ] **Notes** - Any pending tasks in memory files?

## Track your checks

Update `memory/heartbeat-state.json` with timestamps.

## When to Stay Silent

Reply HEARTBEAT_OK when:
- Late night (23:00-08:00) unless urgent
- Nothing new since last check
- Just checked < 30 minutes ago
EOF
```

### Create PERMISSIONS.md

```bash
cat > PERMISSIONS.md << 'EOF'
# PERMISSIONS.md - Sensitive Data Approvals

Track which services have been approved to receive your sensitive information.

## Approval Levels

- **One-time**: Valid for single action only, re-ask next time
- **Forever**: Remembered until manually revoked

## Approved Services

| Service | Data Type | Approval | Date | Notes |
|---------|-----------|----------|------|-------|
| *Example:* Amazon | Payment | Forever | 2026-02-20 | Primary shopping |

## Denied Services

| Service | Data Type | Date | Reason |
|---------|-----------|------|--------|
| *Example:* SketchySite | Payment | 2026-02-20 | Not trusted |

## How to Revoke

To revoke a "Forever" approval, delete the row from the table above.
The agent will ask again next time that service is used.

## Audit Log

<!-- Agent will append entries here -->
### Recent Activity

*No activity yet*
EOF
```

### Create Initial Memory File

```bash
# Create today's memory file
cat > "memory/$(date +%Y-%m-%d).md" << 'EOF'
# Memory - Today

## Session Log

<!-- The agent will update this file throughout the day -->

## Decisions Made

## Tasks Completed

## Next Steps
EOF
```

---

## 8. Configure Email Access

### Option A: Himalaya (IMAP/SMTP)

```bash
# Run the interactive setup wizard
himalaya account configure
```

Or create the config manually at `~/.config/himalaya/config.toml`:

```toml
[accounts.personal]
email = "you@example.com"
display-name = "Your Name"
default = true

backend.type = "imap"
backend.host = "imap.example.com"
backend.port = 993
backend.encryption.type = "tls"
backend.login = "you@example.com"
backend.auth.type = "password"
backend.auth.cmd = "security find-generic-password -s 'email-imap' -w"

message.send.backend.type = "smtp"
message.send.backend.host = "smtp.example.com"
message.send.backend.port = 587
message.send.backend.encryption.type = "start-tls"
message.send.backend.login = "you@example.com"
message.send.backend.auth.type = "password"
message.send.backend.auth.cmd = "security find-generic-password -s 'email-smtp' -w"
```

#### Gmail-specific config:

```toml
[accounts.gmail]
email = "you@gmail.com"
display-name = "Your Name"
default = true

backend.type = "imap"
backend.host = "imap.gmail.com"
backend.port = 993
backend.encryption.type = "tls"
backend.login = "you@gmail.com"
backend.auth.type = "password"
backend.auth.cmd = "security find-generic-password -s 'gmail-app-password' -w"

message.send.backend.type = "smtp"
message.send.backend.host = "smtp.gmail.com"
message.send.backend.port = 587
message.send.backend.encryption.type = "start-tls"
message.send.backend.login = "you@gmail.com"
message.send.backend.auth.type = "password"
message.send.backend.auth.cmd = "security find-generic-password -s 'gmail-app-password' -w"
```

> **Gmail Users:** Generate an App Password at https://myaccount.google.com/apppasswords

Store the password securely:

```bash
# macOS Keychain
security add-generic-password -s "gmail-app-password" -a "$USER" -w "your-app-password"

# Linux (pass): Install pass first, then
pass insert email/gmail
```

### Option B: gog (Google Workspace)

```bash
# Step 1: Get OAuth credentials from Google Cloud Console
# Download client_secret.json from https://console.cloud.google.com/apis/credentials

# Step 2: Configure gog
gog auth credentials /path/to/client_secret.json
gog auth add you@gmail.com --services gmail,calendar,contacts

# Verify
gog auth list
```

### Test Email Access

```bash
# Himalaya
himalaya envelope list --page-size 5

# gog
gog gmail search 'newer_than:1d' --max 5
```

---

## 9. Configure Web Browsing

### Get a Brave Search API Key (Free tier available)

1. Go to https://brave.com/search/api/
2. Create an account and generate an API key
3. Choose "Data for Search" plan

### Configure Brave Search

```bash
# Using the CLI wizard (recommended)
cd ~/path/to/openclaw
pnpm openclaw configure --section web
```

Or edit `~/.openclaw/openclaw.json` directly:

```json
{
  "tools": {
    "web": {
      "search": {
        "enabled": true,
        "provider": "brave",
        "apiKey": "YOUR_BRAVE_API_KEY"
      },
      "fetch": {
        "enabled": true
      }
    }
  }
}
```

### Alternative: Perplexity API (AI-powered search)

For AI-synthesized answers with citations:

```json
{
  "tools": {
    "web": {
      "search": {
        "enabled": true,
        "provider": "perplexity",
        "perplexity": {
          "apiKey": "pplx-your-key-here",
          "baseUrl": "https://api.perplexity.ai",
          "model": "perplexity/sonar-pro"
        }
      }
    }
  }
}
```

Or via OpenRouter (supports crypto/prepaid):

```json
{
  "tools": {
    "web": {
      "search": {
        "enabled": true,
        "provider": "perplexity",
        "perplexity": {
          "apiKey": "sk-or-v1-your-key",
          "baseUrl": "https://openrouter.ai/api/v1",
          "model": "perplexity/sonar-pro"
        }
      }
    }
  }
}
```

### Available Perplexity Models

| Model | Description | Best For |
|-------|-------------|----------|
| `perplexity/sonar` | Fast Q&A with web search | Quick lookups |
| `perplexity/sonar-pro` | Multi-step reasoning | Complex questions |
| `perplexity/sonar-reasoning-pro` | Chain-of-thought | Deep research |

---

## 10. Configure Heartbeats (Ollama)

Edit `~/.openclaw/openclaw.json` to use Ollama for heartbeats:

```json
{
  "agents": {
    "defaults": {
      "heartbeat": {
        "every": "60m",
        "model": "ollama/llama3.2:3b",
        "target": "last",
        "prompt": "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        "ackMaxChars": 300,
        "activeHours": {
          "start": "08:00",
          "end": "23:00"
        }
      }
    }
  },
  "providers": {
    "ollama": {
      "endpoint": "http://localhost:11434"
    }
  }
}
```

### Heartbeat Configuration Options

| Option | Description | Example/Default |
|--------|-------------|-----------------|
| `interval` | Seconds between heartbeat checks | `60` = once per minute |
| `provider` | Set to `"ollama"` to use local LLM instead of paid API | `"ollama"` |
| `model` | Any Ollama model | `llama3.2:3b` (fast and capable) |
| `endpoint` | Ollama's local API | `http://localhost:11434` (default) |

### Alternative: Minimal Heartbeat Config (JSON5)

If your config file supports JSON5:

```json5
{
  agents: {
    defaults: {
      heartbeat: {
        interval: 60,           // seconds (60 = 1 minute)
        provider: "ollama",     // use local LLM
        model: "llama3.2:3b",   // fast and tiny model
        endpoint: "http://localhost:11434"
      }
    }
  }
}
```

### Verify Ollama is Running

```bash
# Make sure Ollama is running
ollama serve &

# In another terminal, test the model
ollama run llama3.2:3b "respond with OK"
# Should respond quickly with "OK" or similar
```

---

## 11. Start OpenClaw

### First-Time Setup (Native)

```bash
cd ~/path/to/openclaw

# Run onboarding wizard
pnpm openclaw onboard

# This will:
# - Generate gateway token
# - Set up initial configuration
# - Guide you through provider setup
```

### First-Time Setup (Docker)

```bash
cd ~/path/to/openclaw

# Run the setup script (builds image + runs onboarding)
./docker-setup.sh

# Or manually:
docker build -t openclaw:local .
docker compose run --rm openclaw-cli onboard
```

### Start the Gateway (Native)

```bash
# Development mode (foreground, with logs)
pnpm openclaw gateway run

# Or in background
pnpm openclaw gateway run &

# Check status
pnpm openclaw status
```

### Start the Gateway (Docker)

```bash
# Start in background
docker compose up -d openclaw-gateway

# Check status
docker compose ps
docker compose run --rm openclaw-cli status

# View logs
docker compose logs -f openclaw-gateway
```

### Access the Control UI

The gateway serves a web UI at:
- **Native:** http://localhost:18789/openclaw/
- **Docker:** http://localhost:18789/openclaw/ (same, port is mapped)

---

## 12. Verification & Testing

### Check All Components (Native)

```bash
# 1. Verify Gateway is running
pnpm openclaw status --deep

# 2. Verify Ollama is running and model works
ollama run llama3.2:3b "respond with OK"

# 3. Verify email works
himalaya envelope list --page-size 3
# OR
gog gmail search 'newer_than:1d' --max 3

# 4. Verify web search works (via the agent)
# Send a message: "Search the web for today's tech news"

# 5. Check channel status
pnpm openclaw channels status --probe
```

### Check All Components (Docker)

```bash
# 1. Verify Gateway is running
docker compose ps
docker compose run --rm openclaw-cli status --deep

# 2. Verify Ollama is running (on host)
ollama run llama3.2:3b "respond with OK"

# 3. Verify email works (on host, or map tools into container)
himalaya envelope list --page-size 3
# OR
gog gmail search 'newer_than:1d' --max 3

# 4. Check channel status
docker compose run --rm openclaw-cli channels status --probe

# 5. View gateway logs
docker compose logs -f openclaw-gateway
```

### Test Heartbeat

```bash
# Native
pnpm openclaw message send --message "heartbeat test"
tail -f /tmp/openclaw/openclaw-*.log | grep heartbeat

# Docker
docker compose run --rm openclaw-cli message send --message "heartbeat test"
docker compose logs -f openclaw-gateway | grep heartbeat
```

### Test Email Integration

Send a message to your agent:
```
Check my email inbox for unread messages
```

### Test Web Browsing

Send a message:
```
Search the web for: latest AI news today
```

---

## 13. Troubleshooting

### Ollama Issues

```bash
# Check if Ollama is running
curl http://localhost:11434/api/tags

# Restart Ollama
pkill ollama
ollama serve &

# Re-pull model if corrupted
ollama rm llama3.2:3b
ollama pull llama3.2:3b
```

### Email Issues

```bash
# Test IMAP connection (Himalaya)
himalaya account check

# Test Gmail OAuth (gog)
gog auth list
gog gmail search 'newer_than:1h' --max 1
```

### Web Search Issues

```bash
# Verify API key is set
cat ~/.openclaw/openclaw.json | grep -A5 '"web"'

# Test Brave Search manually (requires curl + key)
# Check the API dashboard at brave.com for usage/errors
```

### Gateway Issues

```bash
# Check gateway health
pnpm openclaw health --json

# Check for errors
pnpm openclaw doctor

# Restart gateway
pkill -f "openclaw-gateway" || true
pnpm openclaw gateway run &
```

### Memory Issues

```bash
# Ensure workspace paths exist
ls -la ~/.openclaw/workspace/
ls -la ~/.openclaw/workspace/memory/

# Check file permissions
chmod -R 755 ~/.openclaw/workspace/
```

### Docker Issues

```bash
# Check if containers are running
docker compose ps

# View gateway logs
docker compose logs -f openclaw-gateway

# Check container health
docker compose exec openclaw-gateway node dist/index.js health --json

# Restart containers
docker compose down
docker compose up -d openclaw-gateway

# Rebuild after code changes
docker build -t openclaw:local .
docker compose up -d --force-recreate openclaw-gateway

# Check Docker network (for Ollama connectivity)
docker compose exec openclaw-gateway curl -s http://host.docker.internal:11434/api/tags

# If host.docker.internal doesn't work (Linux), use host IP:
HOST_IP=$(ip route | grep default | awk '{print $3}')
docker compose exec openclaw-gateway curl -s http://$HOST_IP:11434/api/tags

# Verify volume mounts
docker compose exec openclaw-gateway ls -la /home/node/.openclaw/
docker compose exec openclaw-gateway ls -la /home/node/.openclaw/workspace/

# Check environment variables
docker compose exec openclaw-gateway env | grep OPENCLAW
```

---

## Quick Reference

### Daily Commands (Native)

```bash
# Start everything
ollama serve &                    # Start Ollama
pnpm openclaw gateway run &       # Start Gateway

# Check status
pnpm openclaw status --deep

# Stop everything
pkill ollama
pkill -f "openclaw-gateway"
```

### Daily Commands (Docker)

```bash
# Start everything
ollama serve &                              # Start Ollama (on host)
docker compose up -d openclaw-gateway       # Start Gateway (container)

# Check status
docker compose ps
docker compose run --rm openclaw-cli status --deep

# View logs
docker compose logs -f openclaw-gateway

# Stop everything
pkill ollama
docker compose down

# Run CLI commands
docker compose run --rm openclaw-cli <command>
```

### Useful Paths

| Path | Purpose |
|------|---------|
| `~/.openclaw/openclaw.json` | Main configuration |
| `~/.openclaw/workspace/` | Agent workspace (AGENTS.md, SOUL.md, etc.) |
| `~/.openclaw/workspace/memory/` | Daily memory files |
| `~/.config/himalaya/config.toml` | Email config (Himalaya) |
| `/tmp/openclaw/` | Logs |

### API Keys Summary

| Service | Env Variable | Config Key |
|---------|--------------|------------|
| **Anthropic Claude** | `ANTHROPIC_API_KEY` | Set during onboarding |
| Brave Search | `BRAVE_API_KEY` | `tools.web.search.apiKey` |
| Perplexity | `PERPLEXITY_API_KEY` | `tools.web.search.perplexity.apiKey` |
| OpenRouter | `OPENROUTER_API_KEY` | `tools.web.search.perplexity.apiKey` |
| Ollama | N/A (local) | `providers.ollama.endpoint` |

---

## Complete Configuration Example

Here's a full `~/.openclaw/openclaw.json` combining all features from this guide:

```json
{
  "gateway": {
    "port": 18789,
    "bind": "loopback",
    "auth": {
      "mode": "token"
    }
  },
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "userTimezone": "America/New_York",
      "heartbeat": {
        "every": "60m",
        "model": "ollama/llama3.2:3b",
        "target": "last",
        "prompt": "Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.",
        "ackMaxChars": 300,
        "activeHours": {
          "start": "08:00",
          "end": "23:00"
        }
      }
    }
  },
  "providers": {
    "ollama": {
      "endpoint": "http://localhost:11434"
    }
  },
  "tools": {
    "web": {
      "search": {
        "enabled": true,
        "provider": "brave",
        "apiKey": "YOUR_BRAVE_API_KEY"
      },
      "fetch": {
        "enabled": true
      }
    }
  }
}
```

> **Note:** Replace `YOUR_BRAVE_API_KEY` with your actual key. For Docker, use `http://host.docker.internal:11434` for the Ollama endpoint.

---

## Auto-Start on Boot

### Native (macOS)

The onboarding wizard can install a LaunchAgent automatically:

```bash
# During onboarding, select "Yes" when asked to install the daemon
pnpm openclaw onboard --install-daemon

# Or manually install later
pnpm openclaw daemon install
pnpm openclaw daemon start

# Check status
pnpm openclaw daemon status
```

### Native (Linux)

```bash
# Install systemd user service
pnpm openclaw daemon install
systemctl --user enable openclaw-gateway
systemctl --user start openclaw-gateway

# Check status
systemctl --user status openclaw-gateway
```

### Docker (systemd)

Create `/etc/systemd/system/openclaw-docker.service`:

```ini
[Unit]
Description=OpenClaw Gateway (Docker)
Requires=docker.service
After=docker.service

[Service]
Type=simple
WorkingDirectory=/path/to/openclaw
ExecStart=/usr/bin/docker compose up openclaw-gateway
ExecStop=/usr/bin/docker compose down
Restart=unless-stopped
User=your-username

[Install]
WantedBy=multi-user.target
```

Enable it:

```bash
sudo systemctl daemon-reload
sudo systemctl enable openclaw-docker
sudo systemctl start openclaw-docker
```

### Ollama Auto-Start

```bash
# macOS: Ollama app auto-starts (add to Login Items)
# Or use Homebrew services
brew services start ollama

# Linux: systemd
sudo systemctl enable ollama
sudo systemctl start ollama
```

---

## Security for VPS Deployments

If running on a public VPS, secure your installation:

### Bind to Loopback (Recommended)

Don't expose the gateway publicly. Bind to loopback and use SSH tunnel or Tailscale:

```json
{
  "gateway": {
    "bind": "loopback",
    "port": 18789
  }
}
```

Access via SSH tunnel:

```bash
# From your local machine
ssh -L 18789:localhost:18789 user@your-vps

# Then access http://localhost:18789/openclaw/
```

### Firewall Rules

```bash
# UFW (Ubuntu)
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 11434/tcp  # Ollama (if needed remotely)
# Do NOT allow 18789 unless behind auth
sudo ufw enable

# Check
sudo ufw status
```

### Use Tailscale (Recommended for Remote Access)

```bash
# Install Tailscale
curl -fsSL https://tailscale.com/install.sh | sh

# Authenticate
sudo tailscale up

# Access gateway via Tailscale IP
# http://<tailscale-ip>:18789/openclaw/
```

### Docker Network Isolation

```yaml
# docker-compose.yml - restrict to internal network
services:
  openclaw-gateway:
    ports:
      - "127.0.0.1:18789:18789"  # Only localhost
      - "127.0.0.1:18790:18790"
```

---

## Calendar Integration (gog)

If you installed `gog` for Gmail, you also get Calendar access:

### Setup

```bash
# Already done during email setup:
gog auth add you@gmail.com --services gmail,calendar,contacts
```

### Usage Examples

```bash
# List today's events
gog calendar events primary --from "$(date -I)" --to "$(date -I -d tomorrow)"

# Create an event
gog calendar create primary --summary "Meeting" --from "2026-02-21T14:00:00" --to "2026-02-21T15:00:00"

# List calendars
gog calendar list
```

### Update HEARTBEAT.md

Add calendar checking to your heartbeat tasks:

```markdown
## Things to Check (rotate through these)

- [ ] **Emails** - Any urgent unread messages?
- [ ] **Calendar** - Upcoming events in next 24h?
- [ ] **Notes** - Any pending tasks in memory files?

### Calendar Check Command
`gog calendar events primary --from "$(date -I)" --to "$(date -I -d '+2 days')"`
```

---

## Automated Data Analysis & Email Reports

Set up scheduled data downloads from Google Sheets, Excel/CSV, or web sources, analyze them, and send email reports.

### Google Drive Setup (for Sheets)

`gog` supports Google Drive. Add it during auth:

```bash
# Add Drive to your services
gog auth add you@gmail.com --services gmail,calendar,contacts,drive

# Verify
gog auth list
```

### Download Sheets as CSV

Google Sheets can export as CSV via Drive. Use `gog drive`:

```bash
# List files in Drive
gog drive list

# Download a spreadsheet as CSV (export)
gog drive download <file-id> --format csv -o ~/data/report.csv

# Or download from a public/shared sheet URL directly
curl -L "https://docs.google.com/spreadsheets/d/<SHEET_ID>/export?format=csv" -o ~/data/report.csv
```

### Download Excel/CSV from External Sources

For files from URLs (company dashboards, APIs, etc.):

```bash
# Direct download
curl -L "https://example.com/data/export.csv" -o ~/data/export.csv

# With authentication
curl -L -H "Authorization: Bearer $API_TOKEN" "https://api.example.com/reports/daily.csv" -o ~/data/daily.csv

# Download Excel file
curl -L "https://example.com/reports/monthly.xlsx" -o ~/data/monthly.xlsx
```

### Create Mock Data for Testing

Create sample data to test the workflow:

```bash
# Create a test CSV
cat > ~/data/mock-sales.csv << 'EOF'
date,product,region,units,revenue
2026-02-01,Widget A,North,150,4500
2026-02-01,Widget B,South,200,8000
2026-02-02,Widget A,North,175,5250
2026-02-02,Widget B,East,120,4800
2026-02-03,Widget A,West,90,2700
2026-02-03,Widget B,North,250,10000
EOF
```

Upload to Google Sheets for the full workflow:

```bash
# Upload CSV to Drive
gog drive upload ~/data/mock-sales.csv --name "Mock Sales Data"

# The file can now be opened in Google Sheets
```

### Data Analysis Skill

Create a skill for analyzing CSV data. Save as `~/.openclaw/workspace/skills/analyze-data.md`:

```markdown
# Analyze Data Skill

When asked to analyze data files:

## Supported Formats
- CSV files (.csv)
- Excel files (.xlsx) - convert first with: `ssconvert input.xlsx output.csv`

## Analysis Commands

### Quick Stats (Python)
```bash
python3 << 'PYEOF'
import csv
from collections import Counter, defaultdict
from statistics import mean, median

with open('/path/to/data.csv') as f:
    reader = csv.DictReader(f)
    data = list(reader)

# Basic counts
print(f"Total rows: {len(data)}")
print(f"Columns: {data[0].keys() if data else 'none'}")

# Numeric analysis (adjust field names)
values = [float(row['revenue']) for row in data if row.get('revenue')]
if values:
    print(f"Revenue - Sum: ${sum(values):,.2f}, Avg: ${mean(values):,.2f}, Median: ${median(values):,.2f}")
PYEOF
```

### Using QMD Skill
If QMD skill is installed, generate visualizations:
```bash
# Create analysis notebook
qmd analyze ~/data/mock-sales.csv --output ~/reports/analysis.html
```
```

### Scheduled Data Download & Analysis

Add to your `HEARTBEAT.md` for scheduled execution:

```markdown
## Scheduled Data Tasks

### Every Morning (8 AM heartbeat)
- [ ] Download latest sales data from Google Sheets
- [ ] Run analysis script
- [ ] Email summary report

### Data Download Script
Location: `~/.openclaw/workspace/scripts/daily-report.sh`

Run command:
`bash ~/.openclaw/workspace/scripts/daily-report.sh`

### Report Email
Send to: your-team@company.com
Subject: Daily Sales Report - {date}
```

Create the automation script at `~/.openclaw/workspace/scripts/daily-report.sh`:

```bash
#!/bin/bash
set -e

# Config
SHEET_ID="YOUR_GOOGLE_SHEET_ID"
DATA_DIR="$HOME/data"
REPORT_FILE="$DATA_DIR/daily-report-$(date +%Y-%m-%d).csv"
REPORT_EMAIL="your-email@example.com"

# 1. Download latest data
echo "Downloading data..."
curl -sL "https://docs.google.com/spreadsheets/d/${SHEET_ID}/export?format=csv" -o "$REPORT_FILE"

# 2. Analyze (simple summary)
echo "Analyzing..."
TOTAL_ROWS=$(tail -n +2 "$REPORT_FILE" | wc -l)
TOTAL_REVENUE=$(tail -n +2 "$REPORT_FILE" | cut -d',' -f5 | paste -sd+ | bc)

# 3. Generate report
REPORT_BODY=$(cat << EOF
Daily Sales Report - $(date +%Y-%m-%d)
=====================================

Total Records: $TOTAL_ROWS
Total Revenue: \$$TOTAL_REVENUE

Top products by region attached.

-- Generated by OpenClaw
EOF
)

# 4. Send email (using himalaya)
echo "$REPORT_BODY" | himalaya send -f you@example.com -t "$REPORT_EMAIL" \
  -s "Daily Sales Report - $(date +%Y-%m-%d)" \
  -a "$REPORT_FILE"

echo "Report sent to $REPORT_EMAIL"
```

Make it executable:

```bash
chmod +x ~/.openclaw/workspace/scripts/daily-report.sh
```

### Alternative: Using gog for Email

If using gog instead of himalaya:

```bash
# Send with gog (Gmail)
gog mail send \
  --to "team@company.com" \
  --subject "Daily Sales Report - $(date +%Y-%m-%d)" \
  --body "See attached for today's analysis." \
  --attach "$REPORT_FILE"
```

### Advanced: Python Analysis Script

For more sophisticated analysis, create `~/.openclaw/workspace/scripts/analyze.py`:

```python
#!/usr/bin/env python3
"""Analyze CSV data and generate email report."""

import csv
import sys
from datetime import datetime
from collections import defaultdict
from statistics import mean

def analyze_csv(filepath):
    with open(filepath) as f:
        reader = csv.DictReader(f)
        data = list(reader)
    
    if not data:
        return "No data found."
    
    # Group by field (adjust to your data)
    by_region = defaultdict(list)
    by_product = defaultdict(list)
    
    for row in data:
        revenue = float(row.get('revenue', 0))
        by_region[row.get('region', 'Unknown')].append(revenue)
        by_product[row.get('product', 'Unknown')].append(revenue)
    
    report = []
    report.append(f"# Analysis Report - {datetime.now().strftime('%Y-%m-%d')}\n")
    report.append(f"**Total Records:** {len(data)}\n")
    
    report.append("\n## Revenue by Region\n")
    for region, revenues in sorted(by_region.items(), key=lambda x: -sum(x[1])):
        report.append(f"- **{region}**: ${sum(revenues):,.2f} (avg: ${mean(revenues):,.2f})")
    
    report.append("\n\n## Revenue by Product\n")
    for product, revenues in sorted(by_product.items(), key=lambda x: -sum(x[1])):
        report.append(f"- **{product}**: ${sum(revenues):,.2f} ({len(revenues)} sales)")
    
    return "\n".join(report)

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: analyze.py <csv_file>")
        sys.exit(1)
    
    report = analyze_csv(sys.argv[1])
    print(report)
    
    # Optionally save to file
    with open(f"/tmp/report-{datetime.now().strftime('%Y-%m-%d')}.md", 'w') as f:
        f.write(report)
```

Run it:

```bash
python3 ~/.openclaw/workspace/scripts/analyze.py ~/data/mock-sales.csv
```

### HEARTBEAT.md for Scheduled Reports

Update your heartbeat to include data tasks:

```markdown
# HEARTBEAT.md

## Daily Tasks (Check at 8 AM)

### Data Report Workflow
1. **Download** - Get latest data from Google Sheets
   ```bash
   curl -sL "https://docs.google.com/spreadsheets/d/SHEET_ID/export?format=csv" -o ~/data/daily.csv
   ```

2. **Analyze** - Run Python analysis
   ```bash
   python3 ~/.openclaw/workspace/scripts/analyze.py ~/data/daily.csv > /tmp/report.md
   ```

3. **Email** - Send report
   ```bash
   cat /tmp/report.md | himalaya send -t team@company.com -s "Daily Report $(date +%Y-%m-%d)"
   ```

### Success Response
If all steps complete, respond with: "✅ Daily report sent to team@company.com"

### Failure Response  
If any step fails, respond with error details and retry instructions.
```

### Testing the Workflow

Test each step manually first:

```bash
# 1. Test download
curl -sL "https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID/export?format=csv" -o /tmp/test.csv
cat /tmp/test.csv

# 2. Test analysis
python3 ~/.openclaw/workspace/scripts/analyze.py /tmp/test.csv

# 3. Test email (dry run)
echo "Test report body" | himalaya send --dry-run -t you@example.com -s "Test Report"

# 4. Full workflow
bash ~/.openclaw/workspace/scripts/daily-report.sh
```

### Cron Alternative (System-Level Scheduling)

For guaranteed scheduling outside of heartbeats:

```bash
# Edit crontab
crontab -e

# Add daily report at 8 AM
0 8 * * * /bin/bash ~/.openclaw/workspace/scripts/daily-report.sh >> ~/.openclaw/logs/daily-report.log 2>&1
```

---

## Slack & Discord Notifications

Send report notifications to team channels instead of (or in addition to) email.

### Discord Webhook

1. **Create webhook** in Discord server:
   - Server Settings → Integrations → Webhooks → New Webhook
   - Copy the webhook URL

2. **Send notifications:**

```bash
# Simple message
curl -X POST "$DISCORD_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"content": "📊 Daily report is ready!"}'

# Rich embed with report data
curl -X POST "$DISCORD_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{
    "embeds": [{
      "title": "Daily Sales Report",
      "description": "Report for '"$(date +%Y-%m-%d)"'",
      "color": 5814783,
      "fields": [
        {"name": "Total Revenue", "value": "$12,500", "inline": true},
        {"name": "Orders", "value": "47", "inline": true}
      ],
      "footer": {"text": "Generated by OpenClaw"}
    }]
  }'
```

3. **Upload file to Discord:**

```bash
# Send CSV as attachment
curl -X POST "$DISCORD_WEBHOOK_URL" \
  -F "file=@/path/to/report.csv" \
  -F 'payload_json={"content": "Here is today'\''s report:"}'
```

### Slack Webhook

1. **Create Slack App:**
   - Go to https://api.slack.com/apps
   - Create New App → From scratch
   - Enable Incoming Webhooks
   - Add to Workspace and copy webhook URL

2. **Send notifications:**

```bash
# Simple message
curl -X POST "$SLACK_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{"text": "📊 Daily report is ready!"}'

# Rich formatted message
curl -X POST "$SLACK_WEBHOOK_URL" \
  -H "Content-Type: application/json" \
  -d '{
    "blocks": [
      {
        "type": "header",
        "text": {"type": "plain_text", "text": "📊 Daily Sales Report"}
      },
      {
        "type": "section",
        "fields": [
          {"type": "mrkdwn", "text": "*Total Revenue:*\n$12,500"},
          {"type": "mrkdwn", "text": "*Orders:*\n47"}
        ]
      },
      {
        "type": "context",
        "elements": [{"type": "mrkdwn", "text": "Generated by OpenClaw • '"$(date +%Y-%m-%d)"'"}]
      }
    ]
  }'
```

### Notification Script

Create `~/.openclaw/workspace/scripts/notify.sh`:

```bash
#!/bin/bash
# Send notifications to multiple channels

MESSAGE="$1"
REPORT_FILE="$2"

# Discord
if [ -n "$DISCORD_WEBHOOK_URL" ]; then
  if [ -n "$REPORT_FILE" ] && [ -f "$REPORT_FILE" ]; then
    curl -sX POST "$DISCORD_WEBHOOK_URL" \
      -F "file=@$REPORT_FILE" \
      -F "payload_json={\"content\": \"$MESSAGE\"}"
  else
    curl -sX POST "$DISCORD_WEBHOOK_URL" \
      -H "Content-Type: application/json" \
      -d "{\"content\": \"$MESSAGE\"}"
  fi
fi

# Slack
if [ -n "$SLACK_WEBHOOK_URL" ]; then
  curl -sX POST "$SLACK_WEBHOOK_URL" \
    -H "Content-Type: application/json" \
    -d "{\"text\": \"$MESSAGE\"}"
fi

echo "Notifications sent."
```

Usage:

```bash
export DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/..."
export SLACK_WEBHOOK_URL="https://hooks.slack.com/services/..."

bash ~/.openclaw/workspace/scripts/notify.sh "📊 Daily report ready!" ~/data/report.csv
```

---

## Database Integration

Store analysis results in a database for historical tracking and trends.

### SQLite (Local, Zero Setup)

Perfect for single-user setups:

```bash
# Create database
sqlite3 ~/.openclaw/data/reports.db << 'SQL'
CREATE TABLE IF NOT EXISTS daily_reports (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  report_date DATE NOT NULL,
  total_revenue REAL,
  total_orders INTEGER,
  top_product TEXT,
  top_region TEXT,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX IF NOT EXISTS idx_report_date ON daily_reports(report_date);
SQL
```

**Insert data from analysis:**

```bash
# After analysis, insert results
sqlite3 ~/.openclaw/data/reports.db << SQL
INSERT INTO daily_reports (report_date, total_revenue, total_orders, top_product, top_region)
VALUES ('$(date +%Y-%m-%d)', 12500.00, 47, 'Widget A', 'North');
SQL
```

**Query historical data:**

```bash
# Last 7 days
sqlite3 -header -column ~/.openclaw/data/reports.db \
  "SELECT * FROM daily_reports ORDER BY report_date DESC LIMIT 7;"

# Weekly average
sqlite3 ~/.openclaw/data/reports.db \
  "SELECT AVG(total_revenue) as avg_revenue, SUM(total_orders) as total_orders
   FROM daily_reports
   WHERE report_date >= date('now', '-7 days');"
```

### Python with SQLite

Create `~/.openclaw/workspace/scripts/db_utils.py`:

```python
#!/usr/bin/env python3
"""Database utilities for report storage."""

import sqlite3
import os
from datetime import datetime, timedelta
from pathlib import Path

DB_PATH = Path.home() / ".openclaw" / "data" / "reports.db"

def init_db():
    """Initialize database with schema."""
    DB_PATH.parent.mkdir(parents=True, exist_ok=True)
    conn = sqlite3.connect(DB_PATH)
    conn.execute("""
        CREATE TABLE IF NOT EXISTS daily_reports (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            report_date DATE NOT NULL UNIQUE,
            total_revenue REAL,
            total_orders INTEGER,
            avg_order_value REAL,
            top_product TEXT,
            top_region TEXT,
            raw_data TEXT,
            created_at DATETIME DEFAULT CURRENT_TIMESTAMP
        )
    """)
    conn.commit()
    conn.close()

def save_report(date, revenue, orders, top_product, top_region, raw_data=None):
    """Save or update daily report."""
    conn = sqlite3.connect(DB_PATH)
    avg_value = revenue / orders if orders > 0 else 0
    conn.execute("""
        INSERT OR REPLACE INTO daily_reports 
        (report_date, total_revenue, total_orders, avg_order_value, top_product, top_region, raw_data)
        VALUES (?, ?, ?, ?, ?, ?, ?)
    """, (date, revenue, orders, avg_value, top_product, top_region, raw_data))
    conn.commit()
    conn.close()

def get_trend(days=7):
    """Get revenue trend for past N days."""
    conn = sqlite3.connect(DB_PATH)
    conn.row_factory = sqlite3.Row
    cursor = conn.execute("""
        SELECT report_date, total_revenue, total_orders
        FROM daily_reports
        WHERE report_date >= date('now', ?)
        ORDER BY report_date
    """, (f'-{days} days',))
    results = [dict(row) for row in cursor.fetchall()]
    conn.close()
    return results

def get_summary():
    """Get summary statistics."""
    conn = sqlite3.connect(DB_PATH)
    cursor = conn.execute("""
        SELECT 
            COUNT(*) as total_reports,
            SUM(total_revenue) as lifetime_revenue,
            AVG(total_revenue) as avg_daily_revenue,
            MAX(total_revenue) as best_day_revenue,
            SUM(total_orders) as lifetime_orders
        FROM daily_reports
    """)
    result = cursor.fetchone()
    conn.close()
    return {
        'total_reports': result[0],
        'lifetime_revenue': result[1] or 0,
        'avg_daily_revenue': result[2] or 0,
        'best_day_revenue': result[3] or 0,
        'lifetime_orders': result[4] or 0
    }

if __name__ == "__main__":
    init_db()
    print("Database initialized at:", DB_PATH)
    
    # Example: save today's report
    save_report(
        date=datetime.now().strftime('%Y-%m-%d'),
        revenue=12500.00,
        orders=47,
        top_product='Widget A',
        top_region='North'
    )
    
    print("\nLast 7 days:")
    for row in get_trend(7):
        print(f"  {row['report_date']}: ${row['total_revenue']:,.2f}")
    
    print("\nSummary:")
    summary = get_summary()
    print(f"  Total Reports: {summary['total_reports']}")
    print(f"  Lifetime Revenue: ${summary['lifetime_revenue']:,.2f}")
    print(f"  Avg Daily Revenue: ${summary['avg_daily_revenue']:,.2f}")
```

### PostgreSQL (For Teams/Production)

For shared/production deployments:

```bash
# Install PostgreSQL
brew install postgresql@16  # macOS
sudo apt install postgresql  # Linux

# Start service
brew services start postgresql@16

# Create database
createdb openclaw_reports

# Connect and create schema
psql openclaw_reports << 'SQL'
CREATE TABLE daily_reports (
  id SERIAL PRIMARY KEY,
  report_date DATE NOT NULL UNIQUE,
  total_revenue DECIMAL(12,2),
  total_orders INTEGER,
  metrics JSONB,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_report_date ON daily_reports(report_date);
CREATE INDEX idx_metrics ON daily_reports USING GIN(metrics);
SQL
```

**Python with PostgreSQL:**

```bash
# Install driver
pip install psycopg2-binary
```

```python
import psycopg2
import json

conn = psycopg2.connect("dbname=openclaw_reports")
cur = conn.cursor()

# Insert with JSON metrics
cur.execute("""
    INSERT INTO daily_reports (report_date, total_revenue, total_orders, metrics)
    VALUES (%s, %s, %s, %s)
    ON CONFLICT (report_date) DO UPDATE SET
        total_revenue = EXCLUDED.total_revenue,
        total_orders = EXCLUDED.total_orders,
        metrics = EXCLUDED.metrics
""", (
    '2026-02-20',
    12500.00,
    47,
    json.dumps({'by_region': {'North': 5000, 'South': 4500, 'East': 3000}})
))

conn.commit()
cur.close()
conn.close()
```

---

## Data Visualization

Generate charts and graphs for visual reports.

### Install Dependencies

```bash
pip install pandas matplotlib seaborn
```

### Visualization Script

Create `~/.openclaw/workspace/scripts/visualize.py`:

```python
#!/usr/bin/env python3
"""Generate visualizations from CSV data."""

import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
from pathlib import Path
from datetime import datetime
import sys

# Set style
sns.set_theme(style="whitegrid")
plt.rcParams['figure.figsize'] = (12, 6)
plt.rcParams['figure.dpi'] = 150

def load_data(filepath):
    """Load CSV data."""
    df = pd.read_csv(filepath)
    if 'date' in df.columns:
        df['date'] = pd.to_datetime(df['date'])
    return df

def revenue_by_region(df, output_path):
    """Bar chart of revenue by region."""
    fig, ax = plt.subplots()
    
    region_data = df.groupby('region')['revenue'].sum().sort_values(ascending=False)
    colors = sns.color_palette('viridis', len(region_data))
    
    bars = ax.bar(region_data.index, region_data.values, color=colors)
    ax.set_xlabel('Region')
    ax.set_ylabel('Revenue ($)')
    ax.set_title('Total Revenue by Region')
    
    # Add value labels
    for bar, val in zip(bars, region_data.values):
        ax.text(bar.get_x() + bar.get_width()/2, bar.get_height() + 100,
                f'${val:,.0f}', ha='center', va='bottom', fontsize=9)
    
    plt.tight_layout()
    plt.savefig(output_path / 'revenue_by_region.png')
    plt.close()
    print(f"Saved: revenue_by_region.png")

def revenue_trend(df, output_path):
    """Line chart of revenue over time."""
    fig, ax = plt.subplots()
    
    daily = df.groupby('date')['revenue'].sum().reset_index()
    
    ax.plot(daily['date'], daily['revenue'], marker='o', linewidth=2, markersize=6)
    ax.fill_between(daily['date'], daily['revenue'], alpha=0.3)
    
    ax.set_xlabel('Date')
    ax.set_ylabel('Revenue ($)')
    ax.set_title('Daily Revenue Trend')
    ax.tick_params(axis='x', rotation=45)
    
    plt.tight_layout()
    plt.savefig(output_path / 'revenue_trend.png')
    plt.close()
    print(f"Saved: revenue_trend.png")

def product_breakdown(df, output_path):
    """Pie chart of revenue by product."""
    fig, ax = plt.subplots()
    
    product_data = df.groupby('product')['revenue'].sum()
    colors = sns.color_palette('Set2', len(product_data))
    
    wedges, texts, autotexts = ax.pie(
        product_data.values,
        labels=product_data.index,
        autopct='%1.1f%%',
        colors=colors,
        explode=[0.02] * len(product_data)
    )
    ax.set_title('Revenue by Product')
    
    plt.tight_layout()
    plt.savefig(output_path / 'product_breakdown.png')
    plt.close()
    print(f"Saved: product_breakdown.png")

def generate_all_charts(csv_path, output_dir=None):
    """Generate all visualizations."""
    df = load_data(csv_path)
    
    if output_dir is None:
        output_dir = Path.home() / '.openclaw' / 'reports' / 'charts'
    else:
        output_dir = Path(output_dir)
    
    output_dir.mkdir(parents=True, exist_ok=True)
    
    print(f"Generating charts from: {csv_path}")
    print(f"Output directory: {output_dir}")
    print(f"Data shape: {df.shape[0]} rows, {df.shape[1]} columns")
    print()
    
    # Generate charts based on available columns
    if 'region' in df.columns and 'revenue' in df.columns:
        revenue_by_region(df, output_dir)
    
    if 'date' in df.columns and 'revenue' in df.columns:
        revenue_trend(df, output_dir)
    
    if 'product' in df.columns and 'revenue' in df.columns:
        product_breakdown(df, output_dir)
    
    print(f"\nAll charts saved to: {output_dir}")
    return output_dir

if __name__ == "__main__":
    if len(sys.argv) < 2:
        print("Usage: visualize.py <csv_file> [output_dir]")
        sys.exit(1)
    
    csv_file = sys.argv[1]
    output = sys.argv[2] if len(sys.argv) > 2 else None
    
    generate_all_charts(csv_file, output)
```

### Generate Charts

```bash
# Generate all charts
python3 ~/.openclaw/workspace/scripts/visualize.py ~/data/mock-sales.csv

# Custom output directory
python3 ~/.openclaw/workspace/scripts/visualize.py ~/data/sales.csv ~/reports/charts/
```

### Email Report with Charts

Update your report script to include charts:

```bash
#!/bin/bash
# daily-report-with-charts.sh

DATA_FILE="$HOME/data/daily.csv"
CHART_DIR="$HOME/.openclaw/reports/charts"
REPORT_EMAIL="team@company.com"

# 1. Download data
curl -sL "https://docs.google.com/spreadsheets/d/${SHEET_ID}/export?format=csv" -o "$DATA_FILE"

# 2. Generate charts
python3 ~/.openclaw/workspace/scripts/visualize.py "$DATA_FILE" "$CHART_DIR"

# 3. Send email with charts attached
himalaya send \
  -f you@example.com \
  -t "$REPORT_EMAIL" \
  -s "Daily Report with Charts - $(date +%Y-%m-%d)" \
  -b "See attached for today's analysis and visualizations." \
  -a "$DATA_FILE" \
  -a "$CHART_DIR/revenue_by_region.png" \
  -a "$CHART_DIR/revenue_trend.png" \
  -a "$CHART_DIR/product_breakdown.png"
```

### Discord with Chart Images

```bash
# Send chart to Discord
curl -X POST "$DISCORD_WEBHOOK_URL" \
  -F "file=@$HOME/.openclaw/reports/charts/revenue_trend.png" \
  -F 'payload_json={"content": "📈 Daily Revenue Trend"}'
```

---

## Webhook Triggers

Trigger reports on-demand from external systems (CI/CD, Zapier, etc.).

### Simple HTTP Endpoint

Create a webhook listener using Python:

`~/.openclaw/workspace/scripts/webhook_server.py`:

```python
#!/usr/bin/env python3
"""Simple webhook server for triggering reports."""

import http.server
import json
import subprocess
import secrets
import os
from urllib.parse import urlparse, parse_qs

PORT = 9876
SECRET_TOKEN = os.environ.get('WEBHOOK_SECRET', secrets.token_urlsafe(32))

class WebhookHandler(http.server.BaseHTTPRequestHandler):
    def do_POST(self):
        # Verify token
        auth = self.headers.get('Authorization', '')
        if auth != f'Bearer {SECRET_TOKEN}':
            self.send_response(401)
            self.end_headers()
            self.wfile.write(b'{"error": "Unauthorized"}')
            return
        
        # Parse request
        content_length = int(self.headers.get('Content-Length', 0))
        body = self.rfile.read(content_length).decode('utf-8')
        
        try:
            data = json.loads(body) if body else {}
        except json.JSONDecodeError:
            data = {}
        
        path = urlparse(self.path).path
        
        # Route to handlers
        if path == '/trigger/daily-report':
            result = self.trigger_daily_report(data)
        elif path == '/trigger/analysis':
            result = self.trigger_analysis(data)
        elif path == '/trigger/notify':
            result = self.trigger_notify(data)
        else:
            self.send_response(404)
            self.end_headers()
            self.wfile.write(b'{"error": "Unknown endpoint"}')
            return
        
        self.send_response(200)
        self.send_header('Content-Type', 'application/json')
        self.end_headers()
        self.wfile.write(json.dumps(result).encode())
    
    def trigger_daily_report(self, data):
        """Trigger the daily report script."""
        script = os.path.expanduser('~/.openclaw/workspace/scripts/daily-report.sh')
        result = subprocess.run(['bash', script], capture_output=True, text=True)
        return {
            'status': 'success' if result.returncode == 0 else 'error',
            'output': result.stdout,
            'error': result.stderr
        }
    
    def trigger_analysis(self, data):
        """Trigger analysis on a specific file."""
        file_url = data.get('file_url')
        if not file_url:
            return {'status': 'error', 'message': 'file_url required'}
        
        # Download file
        import tempfile
        import urllib.request
        with tempfile.NamedTemporaryFile(suffix='.csv', delete=False) as f:
            urllib.request.urlretrieve(file_url, f.name)
            temp_path = f.name
        
        # Run analysis
        script = os.path.expanduser('~/.openclaw/workspace/scripts/analyze.py')
        result = subprocess.run(
            ['python3', script, temp_path],
            capture_output=True, text=True
        )
        
        os.unlink(temp_path)
        return {
            'status': 'success' if result.returncode == 0 else 'error',
            'analysis': result.stdout,
            'error': result.stderr
        }
    
    def trigger_notify(self, data):
        """Send a notification."""
        message = data.get('message', 'Triggered via webhook')
        script = os.path.expanduser('~/.openclaw/workspace/scripts/notify.sh')
        result = subprocess.run(['bash', script, message], capture_output=True, text=True)
        return {'status': 'success', 'message': 'Notification sent'}
    
    def log_message(self, format, *args):
        print(f"[{self.log_date_time_string()}] {args[0]}")

def main():
    print(f"Starting webhook server on port {PORT}")
    print(f"Secret token: {SECRET_TOKEN}")
    print(f"\nEndpoints:")
    print(f"  POST /trigger/daily-report - Run daily report")
    print(f"  POST /trigger/analysis     - Analyze CSV (pass file_url in body)")
    print(f"  POST /trigger/notify       - Send notification (pass message in body)")
    print(f"\nExample:")
    print(f"  curl -X POST http://localhost:{PORT}/trigger/daily-report \\")
    print(f"    -H 'Authorization: Bearer {SECRET_TOKEN}'")
    
    server = http.server.HTTPServer(('', PORT), WebhookHandler)
    server.serve_forever()

if __name__ == "__main__":
    main()
```

### Run the Server

```bash
# Start server (foreground)
python3 ~/.openclaw/workspace/scripts/webhook_server.py

# Or as background service
nohup python3 ~/.openclaw/workspace/scripts/webhook_server.py > ~/.openclaw/logs/webhook.log 2>&1 &
```

### Trigger from External Systems

```bash
# From CI/CD, Zapier, or any HTTP client:
curl -X POST "http://your-server:9876/trigger/daily-report" \
  -H "Authorization: Bearer YOUR_SECRET_TOKEN"

# Trigger with parameters
curl -X POST "http://your-server:9876/trigger/analysis" \
  -H "Authorization: Bearer YOUR_SECRET_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"file_url": "https://example.com/data.csv"}'

# Send custom notification
curl -X POST "http://your-server:9876/trigger/notify" \
  -H "Authorization: Bearer YOUR_SECRET_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"message": "🚀 Deployment complete!"}'
```

### Systemd Service (Linux)

Create `/etc/systemd/system/openclaw-webhook.service`:

```ini
[Unit]
Description=OpenClaw Webhook Server
After=network.target

[Service]
Type=simple
User=your-username
Environment=WEBHOOK_SECRET=your-secure-token
ExecStart=/usr/bin/python3 /home/your-username/.openclaw/workspace/scripts/webhook_server.py
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable:

```bash
sudo systemctl daemon-reload
sudo systemctl enable openclaw-webhook
sudo systemctl start openclaw-webhook
```

### Secure with Nginx (Production)

For production, put behind Nginx with HTTPS:

```nginx
# /etc/nginx/sites-available/webhook
server {
    listen 443 ssl;
    server_name webhook.yourdomain.com;
    
    ssl_certificate /etc/letsencrypt/live/webhook.yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/webhook.yourdomain.com/privkey.pem;
    
    location / {
        proxy_pass http://127.0.0.1:9876;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### Integration Examples

**GitHub Actions:**

```yaml
# .github/workflows/trigger-report.yml
name: Trigger Daily Report
on:
  schedule:
    - cron: '0 8 * * *'
  workflow_dispatch:

jobs:
  trigger:
    runs-on: ubuntu-latest
    steps:
      - name: Trigger Report
        run: |
          curl -X POST "${{ secrets.WEBHOOK_URL }}/trigger/daily-report" \
            -H "Authorization: Bearer ${{ secrets.WEBHOOK_SECRET }}"
```

**Zapier:**
1. Create a Zap with your trigger (e.g., "New row in Google Sheets")
2. Add Webhooks by Zapier action
3. Configure POST to your webhook URL with Authorization header

---

## Channel Setup (Optional)

Connect messaging platforms to interact with your agent from anywhere.

### WhatsApp

```bash
# During onboarding, or:
pnpm openclaw configure --section channels

# Scan QR code with WhatsApp app
# Your phone number will be allowlisted automatically
```

### Telegram

```bash
# 1. Create a bot with @BotFather
# 2. Get your bot token
# 3. During onboarding, enter the token

# Or manually configure:
pnpm openclaw configure --section channels
```

### Discord

```bash
# 1. Create a Discord application at https://discord.com/developers
# 2. Create a bot and get the token
# 3. Invite bot to your server

pnpm openclaw configure --section channels
```

See full channel docs: `/docs/channels/`

---

## Next Steps

After completing this guide:

1. **Customize your workspace** - Edit USER.md, SOUL.md, and IDENTITY.md to personalize your agent
2. **Set up channels** - Connect WhatsApp, Telegram, Discord, etc. (see `/docs/channels/`)
3. **Explore skills** - Check `skills/` folder for available capabilities
4. **Create automations** - Set up cron jobs and webhooks (see `/docs/automation/`)

---

*Last updated: February 2026*
