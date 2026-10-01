# Whatszara — Desktop Assistant via WhatsApp + LLM

[![Docs](https://img.shields.io/badge/docs-whatszara-docs-25D366?style=flat-square)](https://github.com/Preet3627/whatszara-docs)
[![GitHub](https://img.shields.io/github/license/Preet3627/whatszara?style=flat-square&color=25D366)](LICENSE)

Control your desktop from anywhere using WhatsApp messages. Talk to an LLM through WhatsApp, and it executes your commands — shell, apps, media, file access — with a secure permission system and risk-based approval.

Built on top of **[whatsapp-mcp](https://github.com/lharries/whatsapp-mcp)** by Luke Harries, scaled from a simple MCP server into a full desktop automation platform.

**No Python required.** Everything is compiled into a single Tauri desktop app (Rust) + Go bridge for WhatsApp. No interpreters, no virtualenvs, no pip.

## How It Works

```
WhatsApp Message ──▶ Orchestrator ──▶ LLM (Ollama/Claude/Groq/etc.)
                        │                        │
                        ▼                        ▼
                 Policy/Risk Check          Decides Action + Params
                        │                        │
                        └────────┬───────────────┘
                                 ▼
                    ┌─────────────────────┐
                    │  Risk Assessment     │
                    │  Low → Auto-execute  │
                    │  Med → User Approve  │
                    │  High → User Confirm │
                    └──────────┬──────────┘
                               ▼
                        System Action (shell, apps, media...)
                               │
                               ▼
                        Result sent back via WhatsApp
```

## Features

### ✅ Completed
- [x] Python eliminated — everything in Rust + Go (zero Python dependency)
- [x] Multi-LLM provider abstraction (Ollama, Claude, Groq, Grok, Gemini) in Rust
- [x] Live model list fetching for all 5 providers via their REST APIs
- [x] Tauri desktop app with system tray and 6-tab dashboard
- [x] Policy engine with 3 risk profiles (High/Medium/Low)
- [x] Per-tool permissions (independently toggle shell, file, media, apps, WhatsApp)
- [x] Structured action types with propose → evaluate → execute flow
- [x] WhatsApp account allowlist + per-contact mode (Assistant/Chat/Summarize/Blocked)
- [x] GUI contacts table with search, allowlist toggle, mode dropdown
- [x] Built-in chat view with message history and live 3-second auto-polling
- [x] AI reply capability from chat view with Enter-to-send
- [x] Risk/approval system: AI-triggered tool calls with approve/reject in chat UI
- [x] Pending actions panel with Approve/Reject buttons and badge counter
- [x] Shell command executor with blocklist (disabled by default)
- [x] App launcher, volume control, media playback (macOS)
- [x] Desktop image scanner
- [x] Reversible undo journal for all actions
- [x] Permanent WhatsApp auth via macOS/iCloud Keychain (auto-save + restore)
- [x] Persistent policy config in Keychain (allowlist, modes, permissions)
- [x] Configurable Ollama endpoint from GUI
- [x] API key management for cloud providers from GUI
- [x] API_KEY env var auth on Go bridge endpoints
- [x] Logout button to clear auth and keychain entries
- [x] WhatsApp MCP tools in Rust (SQLite reads + HTTP to Go bridge)
- [x] Setup.sh one-click bootstrap
- [x] Multi-platform CI + Release workflows (GitHub Actions)
- [x] MIT License with whatsapp-mcp attribution

### 📋 Planned
- [ ] reCAPTCHA + image-to-text verification integration
- [ ] Scheduled/automated actions
- [ ] Multiple WhatsApp number support
- [ ] Voice message transcription
- [ ] Plugin system for custom tools

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           Whatszara                                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌────────────────────────┐      ┌─────────────────────────────────┐    │
│  │   WhatsApp Layer        │      │    Tauri Desktop App (Rust)      │    │
│  │   (Go Bridge)           │─────▶│                                  │    │
│  │   - whatsmeow client    │      │  ┌──────────────────────────┐   │    │
│  │   - SQLite msg store    │      │  │  LLM Providers           │   │    │
│  │   - REST API :8080      │      │  │  - Ollama (live fetch)   │   │    │
│  │   - API_KEY auth        │      │  │  - Claude/Groq/Grok/Gem  │   │    │
│  └──────────┬──────────────┘      │  └──────────────────────────┘   │    │
│             │                     │                                  │    │
│             │                     │  ┌──────────────────────────┐   │    │
│   ┌─────────▼──────────┐         │  │  Policy Engine           │   │    │
│   │  SQLite (messages)  │◀────────│  │  - 3 risk profiles      │   │    │
│   │  + contacts table   │         │  │  - Per-tool permissions │   │    │
│   └─────────────────────┘         │  │  - Allowlist + modes    │   │    │
│                                   │  └──────────────────────────┘   │    │
│                                   │                                  │    │
│  ┌────────────────────────┐      │  ┌──────────────────────────┐   │    │
│  │  macOS Keychain         │      │  │  Action Engine          │   │    │
│  │  - WA session (auto)   │      │  │  - Shell (disabled)     │   │    │
│  │  - Config (allowlist   │      │  │  - macOS: osascript     │   │    │
│  │    modes, perms)       │      │  │  - Volume / Media       │   │    │
│  └────────────────────────┘      │  │  - File scanner         │   │    │
│                                   │  │  - Undo journal        │   │    │
│                                   │  └──────────────────────────┘   │    │
│                                   │                                  │    │
│                                   │  ┌──────────────────────────┐   │    │
│                                   │  │  Risk/Approval System    │   │    │
│                                   │  │  - Tool call parsing     │   │    │
│                                   │  │  - Pending actions queue │   │    │
│                                   │  │  - Approve/Reject UI     │   │    │
│                                   │  └──────────────────────────┘   │    │
│                                   │                                  │    │
│                                   │  ┌──────────────────────────┐   │    │
│                                   │  │  Frontend (HTML/JS)      │   │    │
│                                   │  │  - Dashboard + Wizard   │   │    │
│                                   │  │  - Chat view + polling  │   │    │
│                                   │  │  - Permissions table    │   │    │
│                                   │  │  - Provider config      │   │    │
│                                   │  │  - Settings + Keychain  │   │    │
│                                   │  └──────────────────────────┘   │    │
│                                   └─────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────┘
```

## Quick Start

### Prerequisites
- **Go** (for WhatsApp bridge)
- **Node.js 20+** + **Rust** (for Tauri desktop app)
- **FFmpeg** (optional — for audio messages)

**Python is NOT required.**

### Setup (30 seconds)

```bash
chmod +x setup.sh && ./setup.sh
# Or: make setup
```

### Run

```bash
# Terminal 1: WhatsApp bridge
make bridge
# Scan QR code with WhatsApp mobile app → Linked Devices → Link a Device

# Terminal 2: Desktop app
make desktop
```

The app auto-starts the bridge. On first use:
1. **Bridge starts** → QR code shown in terminal → scan with WhatsApp
2. **Session auto-saved** to macOS Keychain — no re-scan on restart
3. **Configure a provider** → Ollama works out of the box, or set API keys
4. **Allowlist your number** → your own JID is `self` (pre-allowed)
5. **Send a message** → e.g. "What's my volume?" or "Open Firefox"

### Configuring LLM Providers

Set environment variables or use the Settings GUI in the app:

```bash
export OLLAMA_ENDPOINT=http://localhost:11434     # Ollama (default)
export ANTHROPIC_API_KEY=sk-ant-...                # Claude
export GROQ_API_KEY=gsk-...                         # Groq
export XAI_API_KEY=...                              # Grok (xAI)
export GEMINI_API_KEY=...                           # Gemini
```

The Ollama endpoint and all API keys can also be set from the Settings tab → "Apply Endpoint" and "Apply API Keys" buttons.

## Policy & Permission System

Whatszara uses a **propose → evaluate → execute** flow with risk-based approval.

### Risk Profiles

| Risk Level | Examples | Approval Required |
|-----------|----------|------------------|
| **Low** | Read volume, list files | None (auto-execute) |
| **Medium** | Open apps, play music, set volume | User approve in chat UI |
| **High** | Shell commands, delete, install | User approve in chat UI |

### Per-Tool Permissions

| Category | Default | Actions |
|----------|---------|---------|
| Shell | **Disabled** | `execute_shell`, `run_command` |
| File Access | Enabled | `list_files`, `list_images`, `get_desktop_paths` |
| Media Control | Enabled | `get_volume`, `set_volume`, `play`, `pause` |
| App Launching | Enabled | `open_app` |
| WhatsApp | Enabled | `send_message`, `search_contacts` |

### Contact Modes

Every allowed contact has a mode:
- **Assistant** — Full AI control (tool calls + approve/reject)
- **Chat** — Text only, no desktop actions
- **Summarize** — 2-3 sentence summary (default)
- **Blocked** — Ignored at policy level

## Persistent Storage

| What | Where | How |
|------|-------|-----|
| WhatsApp session | macOS Keychain (`whatszara-wa-session`) | Auto-saved on first connect, auto-restored on launch |
| Policy config | macOS Keychain (`whatszara-config`) | Auto-saved on every change, manual load from Settings |
| App settings | Browser localStorage | Endpoint URLs, API keys |

## Chat View & AI Replies

The built-in chat view features:
- **Left panel**: Searchable contact list sorted by allowlisted status (allowlisted contacts first)
- **Right panel**: Message history with timestamps, auto-scroll to newest
- **3-second auto-polling** for live updates
- **Reply area**: Type a message, AI processes and sends response via WhatsApp. Only visible for allowlisted contacts
- **Pending actions panel**: Shows AI-triggered tool calls with Approve/Reject buttons. High-risk actions require approval before execution

## Keychain Integration

- **WhatsApp auth**: Session DB bytes are base64-encoded and stored via `security add-generic-password`. Restored on app startup — no QR re-scan needed
- **Policy config**: Allowlist, contact modes, and tool permissions are serialized to JSON and stored separately. Auto-saved on every change
- **Logout**: Kills the bridge, deletes both Keychain entries, removes session file. Click "Logout & Disconnect" on the Dashboard

## License

MIT Licensed. © 2026 Latestinssan. The WhatsApp bridge incorporates code from [whatsapp-mcp](https://github.com/lharries/whatsapp-mcp) by Luke Harries.

## Documentation

Full docs site: **[github.com/Preet3627/whatszara-docs](https://github.com/Preet3627/whatszara-docs)**
