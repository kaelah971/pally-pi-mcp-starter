# Pally × Pi MCP Starter

> **Use case:** Connect your Pi coding agent to Pally, so you can access your personal assistant's existing context without leaving your development environment.

A minimal, reproducible starter that connects [Pi](https://pi.dev/docs/latest/mcp) (coding agent) to [Pally's MCP server](https://pally.com/mcp) and proves read-only cross-client retrieval — e.g., asking from Pi about something you previously saved through Pally on iMessage.

No extra packages. No MCP bridge. Just Pi + Pally's official endpoint.

---

## Contents

- [How it works](#how-it-works)
- [Requirements](#requirements)
- [Tested setup](#tested-setup)
- [Authenticate](#authenticate)
- [First working request](#first-working-request)
- [Tools](#tools)
- [Troubleshooting](#troubleshooting)
- [Experiment evidence (sanitized)](#experiment-evidence-sanitized)
- [Security](#security)
- [Project layout](#project-layout)
- [What this starter does not do](#what-this-starter-does-not-do)
- [References](#references)

---

## How it works

```text
┌─────────────┐      MCP (Streamable HTTP + OAuth)      ┌────────────────────┐
│  Pi agent   │ ─────────────────────────────────────▶ │ agent.pally.com    │
│  (this dev  │ ◀───────────────────────────────────── │ /mcp               │
│  env)       │   message_pally / check_pally /        │  Pally assistant   │
└─────────────┘   start_call (not used here)           └────────────────────┘
       │                                                        ▲
       │ .pi/mcp.json                                          │ iMessage, X,
       │ {"pally":{"url":                                      │ email, calendar,
       │   "https://agent.pally.com/mcp"}}                     │ reminders, memory
       ▼                                                        │
 Local project files                                    Prior context lives
 (code + docs)                                          with Pally already
```

You ask Pi in plain words. Pi delegates to Pally via `message_pally`. Pally returns consolidated memory (stage, channels, reminders). You stay in your development environment.

> Verified topology during development: `pi mcp list --json` showed `pally` as `scope: project`, `source: .pi/mcp.json`, `exposure: codemode`, `transport: https://agent.pally.com/mcp`, `state: connected`.

---

## Requirements

- **Pi 1.3.0** (tested — `pi --version → 1.3.0`). Newer Pi versions may behave differently; see [Pi MCP docs](https://pi.dev/docs/latest/mcp) if commands or output differ.
- A **Pally account** with existing context (e.g., something saved via iMessage).
- Network access to `https://agent.pally.com/mcp` (OAuth in browser).
- This repo checked out locally. No `npm install`, no bridge server, no API keys in files.

Docs consulted:

- `https://pi.dev/docs/latest/mcp` — project vs user config, `pi mcp add -l`, `pi mcp list`, OAuth, `/mcp` TUI.
- `https://pally.com/mcp` — Pally's MCP overview. The published page may show a different tool set than what Pi discovers (see [Tools](#tools)).

---

## Tested setup

Commands below follow Pi's official MCP documentation and were validated on Pi 1.3.0.

### 1. Verify Pi

```bash
pi --version
# → 1.3.0 (tested; newer versions may differ)
```

### 2. Add Pally (project-scoped)

If `.pi/mcp.json` does not already exist, create it from the sanitized example:

```bash
# Option A — copy the example (recommended for this starter)
cp mcp.example.json .pi/mcp.json

# Option B — official CLI pattern (see https://pi.dev/docs/latest/mcp)
pi mcp add -l pally --url https://agent.pally.com/mcp
```

The resulting project config is intentionally minimal — **URL only, no secrets**:

```json
{
  "mcpServers": {
    "pally": {
      "url": "https://agent.pally.com/mcp"
    }
  }
}
```

> Full file: [`mcp.example.json`](./mcp.example.json). A matching [`.pi/mcp.json`](./.pi/mcp.json) with the same safe content is included for direct use.

### 3. Trust the project (first run)

Pi reads `.pi/mcp.json` only after project trust is granted. Approve when Pi prompts on startup.

### 4. Verify connection state

```bash
pi mcp list
```

Expected (observed during testing):

```text
pally: connected, 3 tools (codemode, project)
  https://agent.pally.com/mcp
  tools: message_pally, check_pally, start_call
  offers resources
```

JSON form (`pi mcp list --json`, observed) additionally reports:

```json
{
  "name": "pally",
  "scope": "project",
  "exposure": "codemode",
  "transport": "https://agent.pally.com/mcp",
  "state": "connected"
}
```

If sign-in is required, see [Authenticate](#authenticate). Inside an interactive Pi session you can also use `/mcp` to inspect, reconnect, or change exposure; run `/reload` after editing config outside the session.

---

## Authenticate

Pally uses OAuth. Credentials are stored **outside the repo** (for example, under your Pi agent home directory in `mcp-auth.json`) — never commit that file.

```bash
pi mcp login pally
```

- Opens a browser sign-in (default timeout 300s).
- On success, `pi mcp list` reports `pally: connected`.
- The connection persisted across fresh `pi` processes during testing (a new shell running `pi mcp list` still showed `connected` without re-login).
- Sign out (only if you intend to re-authenticate): `pi mcp logout pally`.

Full steps and mismatch notes: [`TROUBLESHOOTING.md`](./TROUBLESHOOTING.md).

---

## First working request

Full fictional walkthrough: [`examples/first-retrieval.md`](./examples/first-retrieval.md).

**You ask (read-only intent — adapt names to your own Pally memory):**

```text
What is the current status of my Bluebird Robotics opportunity, and when is my next follow-up reminder? Read-only — do not send, schedule, or outreach.
```

> Fictional opportunity used on purpose. No private application data is reproduced in this starter.

**How Pi sends it (codemode):**

```js
await tools.mcp__pally__message_pally({
  text: "What is the current status of my Bluebird Robotics opportunity, and when is my next follow-up reminder? Read-only — do not send, schedule, or outreach."
})
```

**Expected outcome (shape, EXAMPLE — fictional, not an actual Pally response):**

```text
[EXAMPLE — fictional illustration only, not an actual Pally response]

Your Bluebird Robotics outreach is in stage "researching".
- Saved via iMessage, no outreach sent yet.
- Open item: draft a 3-line intro.
- Next reminder: fictional — Monday Nov 3 at 09:00 (local).
```

**What a tested real run returned (sanitized summary):**

- `status: "complete"` via `message_pally` — `check_pally` was not needed.
- Reported wall time: **~20.7s** (single sample, not a benchmark).
- Content type: outreach stage + channel summary + proof-of-work items + follow-up reminder time, assembled from previously stored cross-client (iMessage-originated) context.
- Private names, email bodies, and account details are omitted by design.

> Clearly distinguish: `examples/first-retrieval.md` = EXAMPLE/fictional. The sanitized summary above describes the shape of a real tested run without reproducing private data.

---

## Tools

Discovered via Pi tool discovery and `pi mcp list --json` during testing on Pi 1.3.0. Only observed behavior is documented. `start_call` is listed but **not exercised** in this starter — no phone calls.

| Tool | Purpose | Input shape (observed) | Notes |
|------|---------|------------------------|-------|
| `message_pally` | Ask Pally to answer or do something using messages, email, calendar, contacts, reminders, memory, calls, bookings, purchases | `{ text: string (1–8000 chars) }` — user words verbatim, or plain-words task with all details | Returns `{ status, reply, content[] }`. If `status: "working"`, poll with `check_pally`. Used in tested run. |
| `check_pally` | Collect the reply to a still-working turn, or the recap of a `start_call` | `{ turn_id?: string }` — omit to check newest message turn | Same presentation rule as `message_pally`. Not needed when `status: "complete"`. |
| `start_call` | Start a live voice call with Pally; returns a call link; recap via `check_pally` | No input shape exercised in this starter | **Do not use for this starter.** Listed for completeness only. |

> Documentation note: the currently published Pally documentation may show a different tool set (for example, two tools: `message_pally` and `check_pally`). Pi discovery during testing surfaced three tools, including `start_call`. This starter documents all three as discovered but exercises only the read-only text path. If your `pi mcp list` shows a different set, follow what your Pi reports.

---

## Troubleshooting

Short version here; full page: [`TROUBLESHOOTING.md`](./TROUBLESHOOTING.md).

### Redirect URI mismatch (observed once, cause not established)

**Observed:** one initial OAuth attempt reported a redirect URI mismatch before a later `pi mcp login pally` succeeded. Exact URIs/ports are omitted.

This starter does **not** claim Pally caused the mismatch, nor that `pi mcp login pally` is a universal fix. A single observation cannot prove a Pally bug, a Pi bug, or the only workaround.

**Steps that worked during the test:**

```bash
pi mcp login pally
pi mcp list
# expect: pally: connected, 3 tools (codemode, project)
```

### Verify without calling Pally

```bash
pi --version
pi mcp list
pi mcp list --json
```

- `pi mcp list` exits non-zero if an entry is invalid or an enabled server is not connected (per Pi docs).
- In-session: `/mcp` shows the full connection error; `/reload` picks up config edits made outside the session.

---

## Experiment evidence (sanitized)

No codes, tokens, session IDs, email bodies, or private account details included.

| Claim | Status |
|-------|--------|
| OAuth via `pi mcp login pally` → `pally: connected` | ✅ Observed (`pi mcp list` + `--json`) |
| 3 tools discovered: `message_pally`, `check_pally`, `start_call` | ✅ Observed (tool discovery + `pi mcp list --json`) |
| Read-only cross-client retrieval (iMessage-originated context → Pi) | ✅ Observed (`message_pally`, `status: complete`) |
| Reported completion time ~20.7s | ✅ Observed (single sample; not a benchmark) |
| Connection remained authenticated in fresh Pi process | ✅ Observed (new shell `pi mcp list` still `connected`) |
| Redirect mismatch root cause | ❓ Observed once, cause not established |
| Async `working` → `check_pally` polling | ⏳ Not tested in this starter |
| Auth durability / logout-login recovery | ⏳ Not tested in this starter |
| Write or delegation behavior | ⏳ Not tested; starter is read-only by convention |

**Observation vs assumption:** rows marked ✅ are observed outputs from the tested run. Latency generalization, mismatch cause, and write-safety are assumptions until further tested.

---

## Security

**Authorization:** Browser OAuth via `pi mcp login pally`. Pi stores credentials outside this repo. This repo contains only the public endpoint URL.

**Permission boundaries:** `message_pally` can reach messages, email, calendar, contacts, reminders, memory, calls, bookings, purchases via Pally. This starter uses **read-only prompts only** (“do not send / schedule / outreach”) — that is a *prompt convention*, not a server-enforced read-only flag. Do not assume writes are blocked.

**Credential safety:**

- Never commit OAuth credential files, `.env`, tokens, OAuth codes, or log excerpts containing secrets.
- Never paste email bodies, private contact details, or application internals into public docs — use fictional examples (as this starter does).
- Prefer project config with URL only. Keep any personal or credential-bearing servers in your user-level Pi config, per Pi docs.

**`.gitignore`:** this starter ignores credential stores, logs, env files, OS/editor noise, and private artifacts while allowing the safe example + project URL config. Before committing, run:

```bash
grep -rEi "refresh_token|client_secret|-----BEGIN|ghp_|sk-[A-Za-z0-9]{10,}" --exclude-dir=node_modules --exclude-dir=.git . || echo "no credential patterns"
```

---

## Project layout

```text
.
├── README.md                  # this starter (you are here)
├── mcp.example.json           # sanitized project config → copy to .pi/mcp.json
├── .pi/
│   └── mcp.json               # project config (URL only, safe to commit)
├── examples/
│   └── first-retrieval.md     # fictional first-request walkthrough (EXAMPLE)
├── TROUBLESHOOTING.md         # tested auth steps + mismatch note
└── .gitignore                 # credential + private-artifact guards
```

---

## What this starter does not do

- No phone calls (`start_call` documented only).
- No sends, schedules, emails, or outreach.
- No Pally account modifications.
- No fabricated benchmarks or untested polling/re-auth claims.
- No GitHub publish by default (validate locally first).

---

## References

- Pi MCP docs: `https://pi.dev/docs/latest/mcp` (setup, project vs user config, `pi mcp add/list/login`, `/mcp`, OAuth, diagnostics)
- Pally MCP: `https://pally.com/mcp` (official overview; tool set may differ from Pi discovery — see [Tools](#tools))
- Endpoint used here: `https://agent.pally.com/mcp`
