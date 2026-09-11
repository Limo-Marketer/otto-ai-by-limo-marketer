# Installing the Otto AI Cowork plugin

This guide takes a clean checkout to a working plugin inside Claude Cowork. It
assumes no prior context — if you're picking this up from someone else, start
here.

**What the plugin is:** the **LimoAnywhere** half of Otto AI. It runs *on the
operator's machine* as a bundled MCP server, signed in as their own
LimoAnywhere user, strictly read-only — trip calendar, quotes, reservations,
booked revenue.

**What the plugin is not:** GoHighLevel. That half stays a **separate hosted
MCP connector** on Limo Marketer's server (Railway), added by URL
(`https://miles-ai-production.up.railway.app/mcp`) and authenticated through
Booked Rides. The analysis and the write-approval gate live there. Operators
can use the plugin with or without it.

---

## 1. Prerequisites

| Requirement | Notes |
|---|---|
| **Node.js 18+** | `node --version`. Needed to build, and the installed plugin currently uses the machine's `node` (Cowork launches the server on the host with the PATH's node). Without it the `otto-limoanywhere` connector cannot start at all. **Clients don't install this themselves:** `/otto-setup`'s preflight installs it for them — in-chat on macOS (Homebrew or nvm, no admin password), a single `winget` PowerShell paste on Windows (the chat's shell there is a sandbox and can't touch the host). |
| **Git** | `git --version`. Cowork installs and updates marketplace plugins via git — without it the marketplace add fails and updates never arrive. `/otto-setup` fixes this too: macOS `brew install git` or `xcode-select --install` triggered from the chat; on Windows it's bundled into the same `winget` paste as Node. |
| **Claude Cowork** | Any paid plan (Pro, Max, Team, Enterprise). Cowork is not available on Free. |
| **Cowork must run LOCALLY** | **Set this at install time, before anything else.** The bundled LimoAnywhere server cannot run in cloud sessions — skills appear but the `la_*` tools don't. Personal (Pro/Max) accounts: Claude Desktop → Settings → Cowork → turn **off** "Run new tasks in the cloud", then fully restart the app. Team/Enterprise: an org admin sets execution mode under Organization Settings → Cowork. This is a Claude app setting — the plugin cannot set it programmatically, but `/otto-setup` and `/otto-doctor` detect the cloud-session symptom and walk the operator through flipping it. |
| **Latest Claude Desktop** | Cowork requires it. |
| **Windows only:** Virtual Machine Platform enabled | Cowork's requirement. |
| **A LimoAnywhere login** | Company ID, username, password — the three fields from the manage.mylimobiz.com form. **Use a dedicated view-only "Otto AI" user**, not an admin login. |

---

## 2. Build the package

```bash
npm install
npm run plugin:build
```

`plugin:build` compiles the TypeScript, bundles the LimoAnywhere server and
its runtime dependencies into a single file
(`plugins/otto-ai-plugin/server/laServer.mjs`), then **starts the packaged server
to verify it runs standalone**. (A single bundled file rather than a vendored
`node_modules`: Cowork's upload validator rejects zip paths containing `@`,
which npm's scoped package directories use.) You should see:

```
Verifying the packaged server starts standalone...
  ok — 11 LimoAnywhere tools

Plugin package ready: plugins/otto-ai-plugin
```

If that verification fails, stop — the package won't work in Cowork either.

Optionally confirm the checkout is healthy first. All five gates pass with no
`.env` and no network access:

```bash
npm run smoke          # stdio server boots, registers tools, degrades gracefully
npm run smoke:la       # the local LimoAnywhere server specifically
npm run smoke:http     # full hosted OAuth flow against an in-memory mock
npm run test:isolation # two-tenant isolation + read-only audit
npm run test:la-creds  # LimoAnywhere credential linking
```

---

## 3. Install into Cowork

**Clients install from the marketplace** — one time, updates flow
automatically:

1. Claude Desktop → **Customize → Plugins → Add marketplace** →
   `Limo-Marketer/otto-ai-by-limo-marketer`.
2. Install **Otto AI**.

**For local testing of an unreleased build**, upload a zip instead. The
contents must sit at the **root** of the archive (not wrapped in a
`otto-ai-plugin/` folder) — the upload dialog accepts `.zip` only:

```bash
cd plugins/otto-ai-plugin && zip -r ../../otto-ai-plugin.zip . -x "*.DS_Store"
```

After installing, open the plugin. You should see:

- **Skill:** `limoanywhere`
- **Connector:** `otto-limoanywhere` (local)
- **Commands:** `/otto-setup`, `/otto-doctor`, `/otto-update`

---

## 4. Connect LimoAnywhere

Run:

```
/otto-setup
```

Claude calls the plugin's `la_connect_start` tool and gives you a one-time
link like `http://127.0.0.1:PORT/setup?token=…`. Open it in a browser on this
machine and enter your company ID, username, and password — a normal login
form, served by the plugin itself. It verifies the login with LimoAnywhere
and — only if it's accepted — saves it to a file on your machine, readable
only by your user. It takes effect immediately: no restart, no new session.

The password never enters the chat and is never sent to Limo Marketer — it
goes browser → local plugin process → file, all on this machine. The link
expires after 10 minutes or one successful connection. (A fallback tool,
`la_connect`, accepts the login as chat input for machines with no browser;
Claude will warn you before using it.)

---

## 5. Verify

```
/otto-doctor
```

Healthy looks like: LimoAnywhere logged in and reading quotes and the
calendar. Then try:

| Ask | Exercises |
|---|---|
| "What's on the calendar today?" | calendar JSON endpoint |
| "How many quote requests came in last week?" | list scraping |
| "How much is booked for next month?" | revenue summary |
| "Which quotes didn't convert?" | quote conversion report |

## 6. Updates

Pushing a release to the marketplace repo is **not** enough for clients to
receive it: Claude Desktop keeps auto-update **off by default for third-party
marketplaces**, and the local marketplace clone
(`~/.claude/plugins/marketplaces/otto-ai`) stays frozen at the install-day
commit until something refreshes it. Since v0.8.0 the plugin handles this
itself:

1. **Update notice + heartbeat** — once per session the server POSTs to the
   hosted `/plugin/heartbeat` endpoint, which answers the version check AND
   records who runs the plugin (`otto_plugin_heartbeats`: one row per
   machine — company ID, host name, platform, version, last seen; never the
   login). Disclosed to the operator in `/otto-setup`; fail-silent, with the
   GitHub manifest as fallback when the hosted server is unreachable. A
   one-line "vX is available" note rides on the first tool result, so Claude
   offers the update unprompted.
2. **`la_update` tool** — installs the newest release in place, on the host,
   on every OS (the chat shell can't touch the host on Windows — that's why
   this is a server tool, not a shell recipe). Works via git when present,
   plain HTTPS otherwise, and never executes what it downloads: the new code
   runs only after a full Claude Desktop restart. `/otto-update` is now just
   this tool plus plain-language reporting.
3. **Automatic updates (opt-in)** — `/otto-setup` offers it once; `la_update
   automatic_updates=on` records it, and from then on the server updates
   itself at session start and says so. The Aug 17 decision doc's rejection
   of *silent* self-updaters holds: explicit opt-in, pinned source
   (`Limo-Marketer/otto-ai-by-limo-marketer`), and every applied update is
   announced in-chat.

Manual paths still work: the operator can enable Claude's own marketplace
auto-update in the plugin manager, or type
`/plugin marketplace update otto-ai`.

**Installs from before the `otto-ai-plugin` rename** (installed as
`miles-ai@otto-ai`) have no update path at all — the name no longer exists in
the marketplace. One-time fix: uninstall the old plugin, reinstall **Otto AI
by Limo Marketer** from the marketplace. The saved LimoAnywhere login
survives (the server reads every previous install variant's credentials
path and migrates the login to the current one on first use).

## 7. Optional: the GoHighLevel connector

The marketing half (leads, conversations, pipeline, the approval-gated
writes) is a separate remote connector, not part of this plugin. Add it in
Claude's connector settings by URL:

```
https://miles-ai-production.up.railway.app/mcp
```

It authenticates through the operator's Booked Rides login. `/otto-doctor`
will include it in its report whenever it's present.

---

## What the first install test found (macOS, Aug 2026)

- **Cowork runs the bundled Node server, but not in a sandbox.** Claude
  Desktop launches it **directly on the host machine**, as the local user,
  using whatever `node` is on the PATH (an nvm install, in the test).
  Implication: a client without Node installed will see the connector fail to
  start; bundling a runtime (per-OS) remains the fix for that.
- **Outbound HTTPS to `manage.mylimobiz.com` works** — the server runs on the
  host, so it logs in from the operator's own IP exactly as designed.
- **Cowork passes the connector's `env` through without defining
  `${CLAUDE_PLUGIN_DATA}`**, so `MILES_LA_CONFIG` arrives as a literal
  unexpanded string. The server treats any value containing `${` as unset and
  falls back to `~/.claude/plugins/data/otto-ai-plugin/la-credentials.json`.
- **Windows (first install, Aug 2026, MSIX/Store build): the connector fails
  with CONNECTION_CLOSED until Node's directory is added to the *system*
  PATH.** Claude Desktop spawns the server on the host (LocalMcpServerManager
  in `main.log`) but with a sanitized ~8-entry PATH that does not include
  `C:\Program Files\nodejs` even when Node is installed and `node --version`
  works in a terminal. Fix: add `C:\Program Files\nodejs` to the **system**
  PATH, fully exit Claude Desktop (tray icon → Exit), relaunch. After that
  the connector started and all ten `la_*` tools registered. Suspected
  Store-build specific (MSIX runs in a container with a sanitized
  environment); the non-Store installer is untested.
- **Windows logs live at `%LOCALAPPDATA%\Claude\logs\main.log`** (not
  `%APPDATA%`); `mcp.log` was empty — the useful spawn detail (the
  `Using MCP server command: node with path:` line) is in `main.log`. On
  MSIX builds, paths the app reports under `AppData\Roaming` are virtualized
  into `AppData\Local\Packages\Claude_*\LocalCache\Roaming`.
- **The shell-based node/git preflight is unreliable on Windows Cowork:**
  the session shell is a Linux sandbox with its own Node, so `node --version`
  there says nothing about the host the server spawns on. Diagnose from
  `main.log`, not the sandbox shell.
- Linux (via Claude Code) remains untested — report findings.

---

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| `plugin:build` says `dist/laServer.js is missing` | Run `npm run build` first, or use `npm run plugin:build`, which does both. |
| Plugin installs but shows no connector | Check `plugins/otto-ai-plugin/server/laServer.mjs` exists. If not, the build didn't finish. |
| Upload rejected: "path with invalid characters" | The zip contains `node_modules` (or other `@`-prefixed paths) from an old build. Re-run `npm run plugin:build` and re-zip. |
| `otto-limoanywhere` fails to start (macOS) | Most likely no `node` on the machine's PATH. |
| `otto-limoanywhere` fails with CONNECTION_CLOSED (Windows) | Node is missing from the PATH **the desktop host uses to spawn servers** — not the same PATH as your terminal's, so `node --version` succeeding proves nothing. Operator-facing fix (scripted into `/otto-setup`): the single `winget` PowerShell paste (installs Node LTS + Git; the MSI sets the system PATH) followed by a computer restart handles most machines; nodejs.org clicked through with defaults is the no-winget fallback; otherwise the one-line PowerShell paste in `/otto-setup` adds `C:\Program Files\nodejs` to the *user* PATH (no admin needed — the host's sanitized PATH was observed to include user-PATH entries). To confirm the diagnosis: the `Using MCP server command: node with path:` line in `%LOCALAPPDATA%\Claude\logs\main.log`. |
| LA tools say "isn't connected yet" | Run `/otto-setup` — its `la_connect_start` page verifies and saves the login, live immediately. The "isn't connected" message itself names the exact path this install reads — start there. Since v0.7.2 the location is **fixed**: `~/.claude/plugins/data/otto-ai-plugin/la-credentials.json` on every install (the plugin no longer sets `MILES_LA_CONFIG`; pre-v0.7.2 releases pointed it at `${CLAUDE_PLUGIN_DATA}`, which moves with the install variant — the root cause of stranded logins). Reads still fall back to every variant path a previous release may have written — `otto-ai-plugin-inline` (zip uploads), pre-rename `miles-ai`/`miles-ai-inline` — newest file first, and self-heal the login to the fixed path. |
| LA says the login was rejected | Wrong credentials, a changed password, or a user without permission for the quotes and calendar screens. |

---

## Repo orientation

If you're working on this rather than only testing it:

- `src/laServer.ts` — the local LimoAnywhere MCP server entrypoint. Registers
  only `LA_TOOLS` + `LA_SETUP_TOOLS`, and deliberately has no GoHighLevel
  credential, vault, or write gate available to it.
- `src/la/` — the LimoAnywhere access layer. **Read-only by construction:**
  every request funnels through `laFetch`, whose `LA_READ_ALLOWLIST` refuses
  anything not on the known read screens. `npm run test:isolation` audits this.
- `src/tools/limoanywhere/` — the eight `la_*` read tools plus the three setup
  tools (`la_connect_start`, `la_connect`, `la_update` — self-update lives in
  the server because it's the only part of the plugin that always runs on the
  operator's host; see `src/la/update.ts`).
- `plugins/otto-ai-plugin/` — the plugin package (skill, commands, manifests).
- `scripts/build-plugin.mjs` — assembles and verifies the package.
- `scripts/release-plugin.mjs` — stages a release into the marketplace repo.
- `.claude/plan/distribution-model-review.md` — why the surfaces split this way.

Read `CLAUDE.md` at the repo root before changing anything. It documents the
invariants the test gates enforce: LimoAnywhere is strictly read-only, every
GoHighLevel write goes through the prepare/confirm approval gate, and no tool
ever accepts a location ID as input.
