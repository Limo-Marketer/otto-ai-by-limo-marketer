---
description: Diagnose the Otto AI setup and report in plain language
---

Check the Otto AI setup and report what's working in plain language. The
operator is not technical: say what's wrong and what to do, not what failed
internally.

1. Check the machine itself. Run in the shell:

   ```bash
   node --version; git --version; uname -s
   ```

   Node.js is what the local LimoAnywhere connector runs on. If anything is
   missing, **fix it for the operator** instead of pointing them at a
   website — the full playbook is `/otto-setup`'s preflight (step 0); follow
   it from here. In short: on a Mac, install Node in this chat (`brew install
   node` if Homebrew exists, else the nvm two-liner) and have them fully quit
   and reopen Claude Desktop; on Windows, give them the single `winget`
   PowerShell paste that installs Node and Git together, then a computer
   restart. Git missing on a Mac: `brew install git`, or run
   `xcode-select --install` and tell them to click Install in the popup —
   updates won't arrive without it, but today's session still works. If both
   are present, just include "machine setup: OK" in the report.

   **If the `la_*` tools are absent from this session entirely**, check the
   causes in `/otto-setup` step 0-A **in order**: first that Cowork is running
   this task locally (Settings → Cowork → "Run new tasks in the cloud" must
   be OFF; org admins control it on Team/Enterprise — the connector can never
   appear in a cloud task), then missing Node.

   **Windows:** this shell is a Linux sandbox, so the check reflects the
   sandbox, not the operator's machine — a passing check does NOT prove the
   host can spawn the connector. If the `la_*` tools are absent on Windows,
   don't diagnose PATHs at the operator; use `/otto-setup`'s escalation path
   (winget paste + restart; then the PATH one-liner), and only collect
   `%LOCALAPPDATA%\Claude\logs\main.log` diagnostics if those fail.

2. Run **`la_check_connection`** — the local LimoAnywhere side. It verifies
   the saved login works and that quotes and the trip calendar can be read.

3. If the hosted Otto AI GoHighLevel connector is also present in this
   session (a `check_connection` tool exists), run it too and include it in
   the report. If it isn't present, don't treat that as a problem — it's a
   separate connector, not part of this plugin.

Then summarize as a short status, one line per system, and give **one** clear
next step if anything is broken. Common cases:

- *The `la_*` tools aren't in this session at all* → either Cowork ran this
  task in the cloud (fix the "Run new tasks in the cloud" setting, restart,
  new task) or Node.js is missing on the machine (install it for them per
  `/otto-setup` step 0, then fully quit and reopen Claude Desktop).
- *LimoAnywhere says it isn't connected* → run `/otto-setup`.
- *Something that was supposedly fixed is still broken* → the plugin may be
  stale. Call the `la_update` tool (that's all `/otto-update` does) and have
  them restart Claude Desktop. If they haven't chosen before, offer automatic
  updates (`la_update` with `automatic_updates: "on"`).
- *A tool result ends with a note that a newer version is available* → relay
  it in one sentence and offer to update right then.
- *LimoAnywhere rejects the login* → the password likely changed. Run
  `/otto-setup` again with current credentials.
- *LimoAnywhere is erroring or timing out* → their system may be down; check
  status.limoanywhere.com and try again shortly.
- *GoHighLevel (if connected) says the login isn't linked* → Limo Marketer
  support needs to finish setup; the operator can't fix this themselves.
- *GoHighLevel (if connected) asks them to sign in* → the connector's
  authorization expired. Reconnect it in Claude's connector settings.

If LimoAnywhere is connected, finish the report with **What you can ask Otto**:
the list below, in plain language, with the example prompts. Keep the grouping
and the examples; adapt the names and places to the operator's business when
you know them (their own vehicles, airports, Conf #s from this session). Skip
the list when LimoAnywhere isn't connected — the fix comes first.

The tool names in brackets are for you, not the operator — don't show them.

### What you can ask Otto

**Look things up** (read-only — nothing in LimoAnywhere changes)

| What | Try asking |
|---|---|
| The calendar [`la_get_schedule`] | "What jobs are on the calendar today?" · "What's booked this weekend?" |
| Quote requests [`la_list_quotes`, `la_get_quote`] | "How many quotes came in yesterday?" · "Any quotes over $1,000 nobody has answered?" · "Show me quote 98394." |
| Reservations [`la_list_reservations`, `la_get_reservation`] | "Pull up reservation 98175." · "Any cancellations this week?" · "Any online bookings we haven't accepted?" |
| Reports [`la_quote_conversion_report`, `la_revenue_summary`] | "Which quotes from last week never turned into bookings?" · "How much is booked for next month?" |

**Make changes** (Otto shows a preview first — nothing changes until you say yes)

| What | Try asking |
|---|---|
| Book a reservation [`la_prepare_reservation`] | "Book Maria Lopez in a Sprinter from MCO to the Hyatt Regency Orlando on Dec 15 at 10am, $250 flat." |
| Create a quote [`la_prepare_quote`] | "Quote John Smith an SUV from Universal Studios to MCO on the 20th at 3pm, $140." |
| Turn a quote into a booking [`la_prepare_quote_conversion`] | "John accepted — convert quote 98410 into a reservation." · "Move quote 98410 to Unfinalized." |
| Change a reservation [`la_prepare_reservation_update`] | "Assign Juan and the Suburban to 98175." · "Cancel 98175." · "Move 98175 to 3pm." · "Make 98175 four passengers." |
| Change the route [`la_prepare_reservation_update`] | "The pickup for 98175 is now the Marriott on International Drive." · "Add a stop at Disney Springs before the drop-off." |
| Add a note [`la_prepare_note`] | "Add a note to 98175: gate code 1234 — hide it from the customer." · "Tell the driver on 98175 to call on arrival." |

Every change is confirmed with `la_confirm_action` only after the operator says
yes to the preview. Say it in one line: *Otto always shows you the change
first, and never emails or texts your customers.*

**What Otto won't do** — do these in LimoAnywhere directly: delete reservations
or quotes (cancelling is fine), take payments, email or text customers, respond
to quotes, or accept online bookings.
