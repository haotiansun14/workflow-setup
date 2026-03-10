# Professional AI Workflow Setup Guide

> **Home Mac**: always-on agent host (get hostname: `hostname`, get Tailscale IP: `tailscale ip -4`)
> **Mobile Mac / iPhone**: thin control surfaces for observation and steering
> **Background**: AI/ML Research Scientist

---

## System Architecture

Your setup follows an **always-on "agent host" + thin "control surfaces"** pattern:

```text
iPhone / Travel Mac
  |- Claude app / claude.ai/code  -> (Remote Control) -> Claude Code session on home Mac
  |- Chat apps (Telegram/Slack)   -> OpenClaw Gateway  -> tools/cron/heartbeat/memory
  +- SSH (Tailscale)              -> tmux / logs / system admin on home Mac

Home Mac (agent host)
  |- Claude Code (+ OMC plugin) running in tmux/terminals
  |- OpenClaw Gateway (launchd supervised)
  |- Tailscale (secure network fabric + file drop + SSH)
  +- Georgia Tech VPN (keeps campus access anchored at home)
```

**The key split:**
- **Claude Code** = "project worker" for repo-centric work (coding, docs, refactors, builds/tests, tool-using agent loop). Supports **Remote Control from browser/mobile** while keeping execution local.
- **OpenClaw** = "personal ops plane" for long-running automation (cron/heartbeat), multi-channel messaging (Telegram/Slack/iMessage), device nodes (including iOS), and memory indexing.
- **Oh-My-ClaudeCode (OMC)** = "orchestrator layer" on top of Claude Code: staged multi-agent pipelines, tmux-based worker panes for Codex/Gemini/Claude, and optional integration to forward events into OpenClaw.

---

## Part 1: Current Setup Inventory

### 1.1 Core AI CLI Tools

| Tool | Install | Purpose |
|------|---------|---------|
| Claude Code | `curl -fsSL https://claude.ai/install.sh \| bash` | Primary AI coding agent |
| Codex CLI | `brew install --cask codex` | OpenAI coding agent |
| Gemini CLI | `npm install -g @google/gemini-cli` | Google AI coding agent |
| CLIProxyAPI | `brew install cliproxyapi` | API proxy for usage tracking |
| OpenClaw | `npm install -g openclaw@latest` | Autonomous personal AI agent |

### 1.2 Brew Packages

```bash
# Formula (essential)
brew install bat cliproxyapi cloudflared expect gcc@12 gh jq openconnect

# Casks
brew install --cask codex swiftbar tailscale
```

### 1.3 Global NPM Packages

```bash
npm install -g @google/gemini-cli
```

### 1.4 Oh My Zsh Setup

```bash
# Plugins
plugins=(git zsh-autosuggestions zsh-syntax-highlighting you-should-use zsh-bat)

# Install plugins (after Oh My Zsh)
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM}/plugins/zsh-syntax-highlighting
git clone https://github.com/MichaelAqworWorker/zsh-you-should-use ${ZSH_CUSTOM}/plugins/you-should-use
git clone https://github.com/fdellwing/zsh-bat ${ZSH_CUSTOM}/plugins/zsh-bat
```

### 1.5 Shell Config (~/.zshrc key sections)

```bash
# Aliases
alias so='source ~/.zshrc'
alias claude-dsp='claude --dangerously-skip-permissions'

# CLIProxyAPI — set your own key (get from cliproxyapi dashboard or config)
export CLIPROXY_API_KEY="<your-cliproxy-api-key>"

# Gemini -> route through CLIProxyAPI for usage tracking
# IMPORTANT: The @google/genai SDK reads GOOGLE_GEMINI_BASE_URL, NOT GEMINI_BASE_URL
export GEMINI_API_KEY="$CLIPROXY_API_KEY"
export GOOGLE_GEMINI_BASE_URL="http://localhost:8317"
export GOOGLE_API_KEY="<your-google-api-key>"
alias gemini='GOOGLE_API_KEY="" command gemini'
```

### 1.6 Claude Code Configuration

**~/.claude/settings.json:**
```json
{
  "model": "opus",
  "statusLine": { "type": "command", "command": "~/.claude/statusline.sh" },
  "enabledPlugins": {
    "oh-my-claudecode@omc": true,
    "document-skills@anthropic-agent-skills": true,
    "huggingface-skills@claude-plugins-official": true,
    "frontend-design@claude-plugins-official": true
  },
  "effortLevel": "high",
  "env": { "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1" }
}
```

**MCP Servers (~/.claude.json):**
```bash
# Context7 — live documentation lookup for any library
claude mcp add -s user context7 -- npx -y @upstash/context7-mcp

# Filesystem — local file access for Claude
claude mcp add -s user filesystem -- npx -y @modelcontextprotocol/server-filesystem ~

# GitHub — repo operations (set GITHUB_PERSONAL_ACCESS_TOKEN in env)
claude mcp add -s user github -- npx -y @modelcontextprotocol/server-github
```

**Skills (~/.claude/skills/):**
- `showdown/` — Pit Claude vs ChatGPT vs Gemini via CLIProxyAPI
- `frontend-slides/` — HTML slide presentations with animations
- `pptx-composer/` — PowerPoint generation

**OMC Config (~/.claude/.omc-config.json):**
```json
{
  "defaultExecutionMode": "ultrawork",
  "team": { "maxAgents": 5, "defaultAgentType": "executor" },
  "setupVersion": "4.7.9"
}
```

### 1.7 CLIProxyAPI + Dashboard (Docker)

```bash
# Clone and setup
git clone https://github.com/itsmylife44/cliproxyapi-dashboard.git
cd cliproxyapi-dashboard

# Create .env (generate secrets)
cat > .env << 'EOF'
JWT_SECRET=$(openssl rand -base64 32)
MANAGEMENT_API_KEY=$(openssl rand -hex 32)
POSTGRES_PASSWORD=$(openssl rand -hex 32)
PERPLEXITY_SIDECAR_SECRET=$(openssl rand -hex 32)
DEPLOY_SECRET=$(openssl rand -hex 32)
WEBHOOK_HOST=http://host.docker.internal:9000
EOF

# config.local.yaml: add your CLIPROXY_API_KEY to api-keys list
# config.local.yaml: add your GOOGLE_API_KEY under gemini-api-key

# OAuth login
cliproxyapi --claude-login
cliproxyapi --codex-login

# Start
docker compose -f docker-compose.local.yml up -d
```

**Key ports (all bound to 127.0.0.1):**
- 3000: Dashboard
- 8317: API Proxy

### 1.8 Networking

**Tailscale:**
```bash
brew install --cask tailscale
# Enable via menu bar, login to tailnet
# Get your Tailscale IP:
tailscale ip -4
```

**GT VPN (~/gtvpn-macos/):**
```bash
brew install openconnect
# Uses openconnect to connect to Georgia Tech VPN
# See your own gtvpn-macos repo (or search GitHub for "gtvpn-macos")
```

**SSH from laptop (via Tailscale):**
```bash
# Get the home Mac's Tailscale IP first:
#   On home Mac: tailscale ip -4
# Then from laptop:
ssh -L 3000:127.0.0.1:3000 -L 8317:127.0.0.1:8317 $(whoami)@<home-mac-tailscale-ip>
```

### 1.9 Key Project Directories

| Directory | Purpose |
|-----------|---------|
| ~/Documents/GitHub/ | Research repos |
| ~/cliproxyapi-dashboard/ | CLIProxyAPI Docker setup |

---

## Part 2: New Mac Bootstrap Script

```bash
#!/bin/bash
# bootstrap-mac.sh — Reproduce entire setup on a new Mac

set -e

# 1. Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 2. Essential brew packages
brew install bat cliproxyapi cloudflared expect gcc@12 gh jq openconnect
brew install --cask codex swiftbar tailscale

# 3. Oh My Zsh + plugins
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
git clone https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# 4. NVM + Node
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.0/install.sh | bash
export NVM_DIR="$HOME/.nvm" && [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
nvm install 22 && nvm use 22

# 5. Global NPM packages
npm install -g @google/gemini-cli

# 6. Claude Code
curl -fsSL https://claude.ai/install.sh | bash

# 7. Miniconda
curl -fsSL https://repo.anaconda.com/miniconda/Miniconda3-latest-MacOSX-arm64.sh -o miniconda.sh
bash miniconda.sh -b -p $HOME/miniconda3 && rm miniconda.sh

# 8. Tailscale — login via menu bar app

# 9. CLIProxyAPI OAuth
cliproxyapi --claude-login
cliproxyapi --codex-login

# 10. OpenClaw (install when ready to use)
# npm install -g openclaw@latest
# openclaw onboard

echo "Done! Now:"
echo "  1. Copy ~/.zshrc from backup"
echo "  2. Copy ~/.claude/ config from backup"
echo "  3. Copy ~/.claude.json (MCP servers) from backup"
echo "  4. Setup Docker + cliproxyapi-dashboard"
echo "  5. Run 'p10k configure' for Powerlevel10k"
```

---

## Part 3: Claude Code Best Practices for Always-On Workflows

### 3.1 Remote Control (iPhone-first, not just SSH)

Claude Code has a first-class **Remote Control** feature: run a local process, then connect from **claude.ai/code** or the **Claude mobile app** while the session continues on your machine (local filesystem/tools/MCP remain available).

```bash
# Start a Remote Control session on home Mac (inside tmux)
claude remote-control --name "Main Workbench"
```

Key constraints:
- Terminal must stay open (tie it to a tmux session)
- One remote connection per Claude Code instance
- Connection uses outbound HTTPS only, routes via Anthropic API over TLS, uses short-lived scoped credentials

### 3.2 CLAUDE.md + Rules for Persistent Memory

For long-lived workflows, Claude Code's persistence is not "keep chatting forever" — it's structured:

- **CLAUDE.md files** (you author): persistent instructions loaded at session start
- **Auto memory** (Claude writes notes from corrections): loaded at start (first 200 lines)
- **`.claude/rules/`**: modular rule files for specific concerns
- **`@path/to/import`**: import stable docs (README, style guide) without inflating CLAUDE.md

Best practices:
```bash
# Generate starting CLAUDE.md for a repo
cd ~/my-project && claude /init

# Keep CLAUDE.md concise (~200 lines max per file)
# Split into modular files:
mkdir -p .claude/rules
echo "Always run tests after modifying source files" > .claude/rules/testing.md
echo "Use snake_case for Python, camelCase for JS" > .claude/rules/style.md
```

### 3.3 Parallel Work with Git Worktrees

Claude Code sessions are directory-scoped. Use **git worktrees** to run parallel sessions across branches:

```bash
# Create worktrees for parallel work
git worktree add ../project-feature-a feature-a
git worktree add ../project-feature-b feature-b

# Run separate Claude sessions in each
tmux new-session -d -s feature-a -c ../project-feature-a 'claude'
tmux new-session -d -s feature-b -c ../project-feature-b 'claude'
```

Inside Claude Code:
- **Background tasks**: press `Ctrl+B` to background a running command while you keep prompting
- **Task lists**: persist across compactions; share across sessions via `CLAUDE_CODE_TASK_LIST_ID`

### 3.4 Hooks as Guardrails (Critical for Always-On)

Hooks run at lifecycle points (`PreToolUse`, `PostToolUse`, `Stop`) and can enforce safety:

```json
// ~/.claude/settings.json — example hooks
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "~/.claude/hooks/block-destructive.sh"
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "command": "~/.claude/hooks/post-edit-verify.sh"
      }
    ],
    "Stop": [
      {
        "command": "~/.claude/hooks/notify-done.sh"
      }
    ]
  }
}
```

**Important**: command hooks run with your full user permissions. Treat them as production automation.

Recommended baseline for an always-on home Mac:
- `PreToolUse` hook that denies destructive patterns (`rm -rf`, dangerous git resets)
- `PostToolUse` hook that runs lightweight verification (formatting, tests) after edits
- `Stop` hook that sends push notification (via ntfy) when Claude finishes or needs input

### 3.5 Plugin Packaging for Repeatable Workflows

Bundle your repeated workflows ("paper digest," "slide builder," "repo audit") as **versioned plugins**:

```text
~/.claude/plugins/my-research-plugin/
  |- skills/
  |   |- paper-digest/SKILL.md
  |   +- slide-builder/SKILL.md
  |- agents/
  |- hooks/
  |- .mcp.json          # MCP servers auto-start when plugin enabled
  +- manifest.json
```

Plugins can include skills, agents, hook configs, scripts, and MCP server definitions.

### 3.6 Scheduling: /loop vs OpenClaw Cron

- **`/loop`** (Claude Code): great for in-session monitoring ("check build status every 5 min"). Session-scoped — stops when terminal closes.
- **OpenClaw cron**: durable schedules persisted to disk, survive restarts. Use for "must run daily even if terminal is closed."
- **`claude -p`** (print mode): can be called from OS-level schedulers (launchd/cron) for one-shot tasks.

---

## Part 4: OMC Orchestration

### 4.1 Team Mode Pipeline

OMC Team mode runs a staged pipeline for large projects:

```
team-plan -> team-prd -> team-exec -> team-verify -> team-fix (loop)
```

The "PRD + verify + fix loop" structure keeps long-running agents from drifting. Already enabled via `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`.

```bash
# Inside Claude Code:
/oh-my-claudecode:team 3:executor "implement feature X"   # spawn 3 parallel agents
/oh-my-claudecode:ralph "complete this task"               # guaranteed-completion loop
/oh-my-claudecode:plan                                     # strategic planning first
```

### 4.2 Tmux Workers for Multi-Model Orchestration

OMC can spawn real tmux panes running `claude` / `codex` / `gemini` CLIs:

```bash
# Inside Claude Code — tri-model consensus:
/oh-my-claudecode:ccg "Review this implementation and suggest improvements"
# -> Sends to all 3 models, Claude synthesizes the best answer

# Ask a specific model:
/oh-my-claudecode:ask codex "Optimize this function for performance"
/oh-my-claudecode:ask gemini "Find relevant research papers on this topic"
```

### 4.3 Observability on an Always-On Box

OMC provides HUD, cost/session reporting, and raw logs under `.omc/state/`. On an agent host, observability is not optional:

- See what's active and where tools are being used
- Estimate cost drift
- Debug "why did it stop?"

### 4.4 Bridge to OpenClaw

OMC documents an **OpenClaw integration**: forwarding Claude Code session events to an OpenClaw gateway. This enables "watch from phone; inject new instructions" — Claude Code runs the heavy project, OpenClaw acts as the always-on notification + routing + scheduling layer.

---

## Part 5: OpenClaw Integration

### 5.1 Architecture: Gateway + Nodes

OpenClaw's architecture centers on a **Gateway WebSocket server**. All clients (CLI, macOS app, iOS/Android nodes) connect over WebSocket and declare roles/scopes.

```text
Home Mac
  +- OpenClaw Gateway (WebSocket server, launchd supervised)
      |- Node: Home Mac (system commands, screen, files)
      |- Node: iPhone (canvas, camera, location, voice)
      +- Channels: Telegram, Slack, iMessage (via BlueBubbles)
```

### 5.2 Install and Configure

```bash
# Install
npm install -g openclaw@latest

# Onboard (connects messaging, sets LLM)
openclaw onboard
# Choose: Claude as LLM backend
# Connect: iMessage (via BlueBubbles) + Telegram for mobile access

# Start daemon (use launchd for persistence)
openclaw up
```

### 5.3 iPhone as a Node

The iOS app connects to Gateway over WebSocket and exposes node capabilities:
- Canvas, screen snapshot, camera, location, voice
- Discovery: LAN Bonjour, Tailscale unicast DNS-SD, or manual host/port
- Pairing: approve device requests on the gateway host

```bash
# Manage device pairing
openclaw devices list
openclaw devices approve <device-id>
```

### 5.4 Cron + Heartbeat for "Always Running"

Two complementary mechanisms:

**Cron jobs** (durable schedules): persisted under `~/.openclaw/cron/`, survive restarts.
```bash
# Example: daily arXiv digest at 07:30
openclaw cron add "daily-arxiv" --schedule "30 7 * * *" \
  --prompt "Search arXiv cs.LG for papers from yesterday, summarize top 5, deliver to Telegram"
```

**Heartbeat** (periodic awareness): runs on an interval, reads `HEARTBEAT.md`, returns `HEARTBEAT_OK` when nothing is needed (suppresses noise).

Use **heartbeat** for lightweight frequent checks ("anything urgent?") and **cron** for deterministic jobs ("daily digest," "hourly inbox triage").

### 5.5 Exec Approvals (Safe Remote Control)

Exec Approvals sit between the agent and real host command execution. Commands run only when policy + allowlist + optional user approval agree.

The killer feature: **forward exec approval prompts to chat channels** and approve with `/approve ...` from your phone (instead of SSHing in).

```bash
# Configure exec approvals
openclaw config set exec.approval_mode "prompt"
openclaw config set exec.forward_channel "telegram"
```

### 5.6 OpenClaw + Claude Code Synergy

- **OpenClaw**: always-on, receives tasks from phone, handles daily automation
- **Claude Code**: deep coding sessions, multi-file refactors, project work
- **Bridge**: OpenClaw triggers Claude Code sessions via shell commands:

```bash
# From phone via OpenClaw:
# "Start a Claude Code session to fix the bug in my-project/backend"
# OpenClaw runs: tmux new-session -d -s fix-bug 'claude "fix the bug in backend/api.ts"'
```

### 5.8 Tailscale Integration

OpenClaw supports first-class Tailscale integration:

```bash
# Expose OpenClaw dashboard on tailnet only (not public)
tailscale serve --bg 3210

# For artifact transfer between devices
# Use Taildrop: tailscale file send <file> <device>
```

Prefer **Tailscale Serve** (tailnet-only) over **Funnel** (public) for personal/work data.

---

## Part 6: End-to-End Workflows

### 6.1 Continuous Massive Projects

Use **Claude Code + OMC** as the execution engine, **OpenClaw** as scheduler/notifier:

1. Run project work in separate directories/worktrees
2. Standardize behavior with `.claude/CLAUDE.md` + `.claude/rules/`
3. Use OMC Team mode for staged execution with verification loops
4. Use `/loop` for in-session monitoring, OpenClaw cron for durable schedules

```bash
# On home Mac — start persistent Claude sessions
tmux new-session -d -s project1 -c ~/project1 'claude'
tmux new-session -d -s project2 -c ~/project2 'claude'

# Resume from SSH (get home IP with: tailscale ip -4)
ssh $(whoami)@<home-mac-tailscale-ip>
tmux attach -t project1

# Or use Remote Control from Claude mobile app — no SSH needed
```

### 6.2 File / Workspace Organization

Two complementary approaches:
- **Claude Code**: for "this repo/directory should be refactored/organized." Permission model confines writes to working directory tree — a safe boundary for large reorganizations.
- **OpenClaw**: for "ongoing organization" — building an index, maintaining a knowledge base, generating periodic digests. Put host commands behind Exec Approvals allowlists.

### 6.3 Email + Messages (Design for "Suggest, Don't Do")

OpenClaw supports multiple chat channels and Gmail integration via webhooks.

**Critical safety note**: A reported incident involved an OpenClaw agent deleting emails rapidly and ignoring stop commands, likely caused by context compaction under load.

Safe operating contract:
- Default to **read + summarize + propose actions**
- Require explicit approval (via Exec Approvals) for destructive actions (delete/archive/bulk changes)
- Keep webhook endpoints behind loopback/tailnet and use dedicated hook tokens

### 6.4 Research: Papers, Web Search, and Digests

**MCP servers for research:**
```bash
# ArXiv paper search and download
claude mcp add -s user arxiv -- uv tool run arxiv-mcp-server --storage-path ~/.arxiv-papers

# ArXiv LaTeX source (critical for ML papers with heavy math)
# See: https://github.com/takashiishida/arxiv-latex-mcp

# RSS feeds (subscribe to arXiv category feeds)
# See: https://github.com/richardwooding/feed-mcp
```

**OpenClaw daily research brief pattern:**
1. Cron job runs `web_search` for targeted queries ("arXiv cs.LG today", "top AI papers last 24h")
2. `web_fetch` on top results (uses Readability extraction, caches 15 min)
3. PDF tool for extraction/summary if papers are linked
4. Composes a short digest + action list
5. Delivers to your preferred channel (Telegram or Slack)

### 6.5 Building HTML/Slides

- **`frontend-slides`** skill: describe aesthetics, get animated HTML presentation (single self-contained file)
- **`pptx-composer`** skill: PowerPoint generation
- **`document-skills`** plugin: PDF, DOCX, XLSX, canvas design

Pattern: write content in Markdown, ask Claude to generate slides from it.

---

## Part 7: Codex + Gemini + Claude Collaboration

### Model Strengths Matrix

| Task | Best Model | How to Use |
|------|-----------|------------|
| Architecture / Planning | Claude Opus | Default model |
| Code generation | Codex / Claude | `/oh-my-claudecode:ccg` for consensus |
| Web search / recent info | Gemini | `gemini` CLI or `/oh-my-claudecode:ask gemini` |
| Paper analysis | Claude + Gemini | `/oh-my-claudecode:ccg` |
| Quick lookups | Gemini Flash | `gemini -m gemini-2.5-flash` |

### Codex CLI Key Features

```bash
# Interactive TUI
codex

# Non-interactive (for scripting/CI)
codex exec "task description"

# Full auto mode
codex --full-auto "task"

# MCP integration
codex mcp add context7 -- npx -y @upstash/context7-mcp

# Expose Codex as MCP server (for Claude Code to call)
codex mcp-server

# Session management
codex resume          # continue last session
codex fork            # branch into new thread
```

**AGENTS.md** for Codex: same concept as CLAUDE.md — project context files loaded automatically.

---

## Part 8: Security & Reliability Guardrails

### 8.1 Claude Code Security

- Permission-based by default (read-only until approved)
- Supports allowlisting safe commands
- Built-in protections: write-scope restriction, sandboxed bash with filesystem/network isolation
- Hooks enforce additional guardrails but run with full user permissions

### 8.2 OpenClaw Security

**Keep it updated** — a high-severity vulnerability allowed malicious websites to take over locally running OpenClaw agents via localhost WebSocket brute-forcing. Fixed in v2026.2.25+.

```bash
# Run security audit after config changes
openclaw security audit
openclaw security audit --deep    # thorough scan
openclaw security audit --fix     # auto-tighten permissions (won't rotate secrets)
```

Minimum baseline:
- Keep gateway behind loopback/tailnet (never expose publicly)
- Use token-based auth, avoid weak password modes
- Put host command execution behind Exec Approvals allowlists
- Run `openclaw security audit` before enabling new channels/skills

### 8.3 Operator Kill Switch

Maintain an emergency access path:
```bash
# SSH into home Mac via Tailscale
ssh $(whoami)@<home-mac-tailscale-ip>

# Stop all agent processes
tmux kill-server                    # kill all tmux sessions
openclaw down                       # stop OpenClaw
docker compose down                 # stop CLIProxyAPI

# Or target specific sessions
tmux kill-session -t <session-name>
```

### 8.4 Remote Access Hygiene

- Enable macOS "Remote Login" (SSH) and scope to specific users
- Prefer Tailscale SSH / Serve over public port exposure
- Use `mosh` for connection resilience on flaky mobile networks:
  ```bash
  brew install mosh    # both client and server
  mosh $(whoami)@<home-mac-tailscale-ip>
  ```

---

## Part 9: Quick Reference

### Daily Workflow Commands

```bash
# Start/resume Claude Code
claude                          # new session
claude --resume <session-id>    # resume previous
claude-dsp                      # skip permissions mode

# Remote Control (access from phone/browser)
claude remote-control --name "Workbench"
# Then visit claude.ai/code from any device

# Persistent background sessions
tmux new -d -s work 'claude-dsp'
tmux attach -t work

# Gemini (routed through CLIProxyAPI)
gemini                          # interactive mode
echo "question" | gemini        # one-shot

# Remote SSH (get home IP: tailscale ip -4)
ssh -L 3000:127.0.0.1:3000 $(whoami)@<home-mac-tailscale-ip>
# Then: http://localhost:3000 for dashboard

# OpenClaw
openclaw up                     # start daemon
openclaw status                 # check status
# Then message via Telegram/iMessage from iPhone
```

### OMC Power Commands (inside Claude Code)

```bash
/oh-my-claudecode:ralph <task>      # Keep working until done + verified
/oh-my-claudecode:team N:executor   # Spawn N parallel agents
/oh-my-claudecode:plan              # Strategic planning
/oh-my-claudecode:ccg <question>    # Tri-model (Claude+Codex+Gemini)
/oh-my-claudecode:ultrawork         # Parallel execution engine
```

### Push Notifications (ntfy)

```bash
# Install ntfy app on phone (iOS/Android)
# Create a hook for Claude Code stop events:
cat > ~/.claude/hooks/notify-done.sh << 'HOOK'
#!/bin/bash
PROJECT=$(basename "$PWD")
curl -s -d "[$PROJECT] Claude Code task complete" \
  -H "Priority: default" \
  "https://ntfy.sh/<your-private-topic>"
HOOK
chmod +x ~/.claude/hooks/notify-done.sh
```

For self-hosted privacy: run ntfy server on home Mac, access via Tailscale.
