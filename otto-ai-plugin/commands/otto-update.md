---
description: Update Otto AI to the latest release from the marketplace
---

Update this plugin to the latest release. The operator is not technical:
report what you did and what to do next in plain language, never raw paths or
JSON unless something needs fixing.

Do this:

1. Call the **`la_update`** tool with no arguments. It runs on this machine
   (inside the plugin's own server, so it works the same on Mac and Windows),
   refreshes the marketplace, and installs the newest release in place. It
   needs no git and asks for nothing.

2. Relay its answer in plain language. The outcomes it can report:
   - **Updated** → tell the operator to **fully quit and reopen Claude
     Desktop** (Mac: Cmd+Q; Windows: system tray icon → Exit) — the new
     version takes effect on restart — then run `/otto-doctor` in a new chat
     to confirm.
   - **Already up to date** → say so, done.
   - **Marketplace not set up / installed by zip / pre-rename install** → the
     tool's message contains the exact fix (usually: Customize → Plugins →
     Add marketplace → `Limo-Marketer/otto-ai-by-limo-marketer`, install **Otto AI
     by Limo Marketer**). Walk them through it; their saved LimoAnywhere
     login survives a reinstall.
   - **Offline** → try again when the machine has internet.

3. If they haven't chosen before, offer **automatic updates** once: "Want
   Otto to keep itself up to date from now on?" If yes, call `la_update` with
   `automatic_updates: "on"`. From then on the plugin updates itself at the
   start of a session and says so — each update still takes effect at the
   next full restart.

**If the `la_update` tool doesn't exist in this session:** the installed
plugin predates self-update (v0.7.x or older), so this one time the update is
manual — and on Windows it can't be done from this chat at all (the shell
here is a sandbox). Mac: run these two lines in the shell, then copy the
refreshed files over the installed copy listed in
`~/.claude/plugins/installed_plugins.json` (key `otto-ai-plugin@otto-ai`):

```bash
git -C ~/.claude/plugins/marketplaces/otto-ai fetch --depth 1 origin main
git -C ~/.claude/plugins/marketplaces/otto-ai reset --hard origin/main
```

```bash
cp -R ~/.claude/plugins/marketplaces/otto-ai/otto-ai-plugin/. "<installPath>/"
```

Windows on an old version: have the operator open **Customize → Plugins**,
uninstall Otto AI, and reinstall it from the marketplace — the saved
LimoAnywhere login survives. Then restart Claude Desktop.
