---
description: Connect Otto AI to LimoAnywhere (login stays on this machine)
---

Set up this operator's LimoAnywhere connection.

The LimoAnywhere tools run locally on this machine. Setup happens on a
one-time browser page served by the plugin itself, so the password is typed
into a normal form on this machine — never into this chat, and never sent to
Limo Marketer.

Do this:

0. **Preflight — check the machine, and fix it yourself where you can.** The
   operator is not technical: never send them off to "go install Node" alone.
   On a Mac you can install what's missing right here in the chat; on Windows
   the fix is a single line they paste. Run this in the shell:

   ```bash
   node --version; git --version; uname -s
   ```

   Then work through whichever of these applies (if node, git, and the
   `la_*` tools are all present, say nothing about the preflight and move on):

   **A. The `la_*` tools are missing from this session entirely.** Two
   possible causes — rule them out in this order:

   1. **Cowork is running this task in the cloud.** The LimoAnywhere
      connector exists only on the operator's machine, so cloud tasks can't
      see it (the skill and this command still appear — that's the telltale
      combination). A giveaway: `uname` says `Linux` but the operator is on a
      Mac. This is a Claude setting — it cannot be changed from this chat, so
      walk them through it:
      - Personal plans (Pro/Max): Claude Desktop → **Settings → Cowork** →
        turn **off** "Run new tasks in the cloud" → fully quit Claude
        (Cmd+Q on Mac; tray icon → Exit on Windows) → reopen, start a **new**
        task, and run `/otto-setup` again.
      - Team/Enterprise: an org admin must set local execution under
        **Organization Settings → Cowork**; the operator can't change it
        themselves.
   2. **Node.js is missing on the machine** — see B (Mac) or C (Windows).

   **B. Mac (`uname` says `Darwin`) and `node` is missing.** The shell runs
   on this machine, so install it for them. Say: "Node.js is missing — it's
   the engine the LimoAnywhere connection runs on. I can install it for you
   right now; it takes a couple of minutes." On their okay:

   - If Homebrew is present (`command -v brew`): `brew install node`.
   - Otherwise, no admin password needed:

     ```bash
     curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
     export NVM_DIR="$HOME/.nvm" && . "$NVM_DIR/nvm.sh" && nvm install --lts
     ```

   Confirm with `node --version`, then have them **fully quit (Cmd+Q) and
   reopen Claude Desktop** and run `/otto-setup` again — the connector only
   sees the new Node on a fresh start.

   **C. Windows.** This shell is a Linux sandbox: its `node`/`git` say nothing
   about the operator's actual computer, and you cannot install anything on
   the host from here. If the `la_*` tools are missing (and A.1 is ruled
   out), give them ONE thing to paste — don't send them to download pages.
   Have them open **PowerShell** (press the Windows key, type "powershell",
   press Enter), paste this single line exactly, and press Enter:

   ```powershell
   winget install -e --id OpenJS.NodeJS.LTS --accept-package-agreements --accept-source-agreements; winget install -e --id Git.Git --accept-package-agreements --accept-source-agreements
   ```

   Tell them what it does in plain language: it installs Node.js and Git
   using Windows' built-in installer; if a "Do you want to allow this app to
   make changes?" box pops up, click **Yes**. When it finishes, **restart the
   computer** and open Claude Desktop again — the restart is what makes it
   take effect. Escalation path if that doesn't get them there:

   1. If PowerShell says `winget` isn't recognized (older machines): go to
      **nodejs.org**, download it, click **Next** through every step without
      changing anything, restart the computer, reopen Claude Desktop.
   2. If Node is installed but the connector still won't start: paste this
      one line in PowerShell, exactly as written:

      ```powershell
      [Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path","User") + ";C:\Program Files\nodejs", "User")
      ```

      Then right-click the Claude icon in the system tray (bottom-right,
      near the clock), choose **Exit**, and reopen Claude Desktop.
   3. Only if all of that fails, collect diagnostics for support: the
      `Using MCP server command: node with path:` line from
      `%LOCALAPPDATA%\Claude\logs\main.log`.

   **D. `git` is missing but node works.** The plugin runs, but marketplace
   updates won't arrive. Fix it now rather than recommending a website:
   - Mac with Homebrew: `brew install git`.
   - Mac without Homebrew: run `xcode-select --install` in the shell — a
     popup appears on their screen; tell them to click **Install** and wait
     for it to finish.
   - Windows: it's already included in the PowerShell paste in C.
   Continue with setup either way — git is not required for today's
   connection, only for updates.

1. Recommend the operator create a **dedicated view-only "Otto AI" user** in
   LimoAnywhere rather than sharing their admin login. It limits what this
   connection can reach and it can be revoked on its own. If they'd rather use
   an existing login, that's their call; don't push twice.

2. Call the **`la_connect_start`** tool. Give the operator the link it returns
   and tell them to open it in a browser **on this machine** and enter their
   company ID, username, and password — the same three fields as the
   manage.mylimobiz.com login form. The page verifies the login with
   LimoAnywhere and saves it (readable only by their user) on this machine.
   The link expires after 10 minutes or one successful connection; if it
   expires, call `la_connect_start` again for a fresh one.

3. When they say they've submitted, run `la_check_connection` and report the
   result in plain language. The connection is live immediately — no restart.
   Mention once, in one sentence: Otto reports its version, the company ID,
   and this computer's name to Limo Marketer so support knows who's on which
   version — never the login itself.

4. **Fallback only** if they truly can't open the page (no browser on this
   machine): the `la_connect` tool takes the three fields as arguments. Warn
   them first that the password will appear in this conversation's history,
   and get their okay before proceeding.

5. Once connected, offer **automatic updates** in one sentence: "Want Otto to
   keep itself up to date automatically?" If yes, call `la_update` with
   `automatic_updates: "on"` — the plugin will then update itself at the
   start of a session and say so (each update takes effect at the next full
   Claude Desktop restart). If they decline, don't ask again; `/otto-update`
   stays available.

If the login is rejected, the likely causes are a typo, a recently changed
password, or a user without permission for the quotes and calendar screens.

Never repeat a password back into the conversation — not in confirmations,
not in summaries.
