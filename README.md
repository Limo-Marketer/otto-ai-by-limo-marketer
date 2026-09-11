# Otto AI Marketplace

Limo Marketer's plugin marketplace for Claude. It currently ships one plugin:

**Otto AI by Limo Marketer** — ask questions about your LimoAnywhere back office: today's
schedule, quote requests, reservations, and booked revenue. It runs on your
own computer, signed in as your own LimoAnywhere user, and is strictly
read-only.

## Install (once)

In **Claude Desktop** (any paid plan):

1. Make sure Cowork runs **on your computer**, not in the cloud: open
   **Settings → Cowork**, turn **off** "Run new tasks in the cloud", then
   fully quit and reopen Claude Desktop. (Otto AI's LimoAnywhere connection
   runs locally on your machine — that's what keeps your login private — so
   cloud sessions can't use it.)
2. Open **Customize → Plugins**.
3. Choose **Add marketplace** and enter:

   ```
   Limo-Marketer/otto-ai-by-limo-marketer
   ```

4. Install **Otto AI by Limo Marketer**.
5. In a session, run `/otto-setup` to connect LimoAnywhere — Claude gives you
   a link to a page on your own computer where you enter your LimoAnywhere
   login. It's saved only on your machine.

Then try: *"What's on the calendar today?"* or *"How much is booked for next
month?"* Run `/otto-doctor` any time to check the connection.

## Updates

When a new version is published here, Claude shows an update for Otto AI
under **Customize → Plugins** — one click, and your saved login is untouched.
