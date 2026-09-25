---
name: limoanywhere
description: Working with a limo operator's LimoAnywhere back office through Otto AI — reading the trip calendar and daily schedule, quote requests and their conversion, reservations across the new/online/unfinalized/deleted screens, and booked revenue; and, with the operator's explicit confirmation, booking a new reservation, changing a reservation's status / driver / car / pickup / rate, or adding a note. Use whenever the question is about trips, jobs, runs, quotes, reservations, confirmation numbers, drivers, vehicles, booked revenue, or booking / assigning / cancelling a trip.
---

# LimoAnywhere operations

These tools work with the operator's LimoAnywhere back office. They run **on
this machine**, signed in as the operator's own LimoAnywhere user — so what you
can see is exactly what they'd see in a browser, and what you can change is
exactly what that user could change.

**Reading is free; changing takes two steps.** Every `la_get_*`, `la_list_*`,
report and check tool is read-only — you never need to ask permission before
reading. Changes go through a *prepare → confirm* gate described under
"Making changes" below: the prepare tool shows a preview and changes nothing,
and only `la_confirm_action`, called after the operator says yes, writes. Never
imply a change was made until `la_confirm_action` has returned "Done".

LimoAnywhere is the *operations* side (actual trips and money). If the
session also has Otto AI's GoHighLevel connector — the *marketing* side
(leads, conversations, pipeline) — many good answers combine both; if it
doesn't, stay on the operations side rather than guessing at marketing data.

## Picking the right tool

| The question | Start with |
|---|---|
| "What's on today / tomorrow / this weekend?" | `la_get_schedule` |
| "How many quotes came in?" | `la_list_quotes` |
| "Tell me about quote 97751" | `la_get_quote` |
| "What reservations do we have?" | `la_list_reservations` |
| "Pull up confirmation 97362" | `la_get_reservation` |
| "Which quotes did we lose?" | `la_quote_conversion_report` |
| "How much is booked this month / next month?" | `la_revenue_summary` |
| "Book a trip for…" / "Put in a reservation" | `la_prepare_reservation`, then `la_confirm_action` |
| "Assign Juan to 98175" / "Cancel 98175" / "Move it to 3pm" | `la_prepare_reservation_update`, then `la_confirm_action` |
| "Add a note to 98175: gate code 1234" | `la_prepare_note`, then `la_confirm_action` |
| Setup or something's broken | `la_check_connection` |

`la_get_schedule` and `la_revenue_summary` both work for **future** dates. "How
much is on the books for next month?" is a normal, answerable question.

## Two quirks that change answers

**Quotes and reservations are filtered by different dates.** `la_list_quotes`
works by *when the quote was requested*. `la_list_reservations` and
`la_revenue_summary` work by *pickup date*. "Quotes last week" and "trips last
week" are different windows over different things — be explicit about which one
you answered, because operators ask for both using the same words.

**Reservations live on four separate screens.** `list_from` picks which:

- `new_reservations` — the main list, and the right default
- `online` — web bookings and eFarm-in that may be *awaiting acceptance*
- `unfinalized` — started but not completed
- `deleted` — removed reservations

A trip missing from the main list may simply be on another screen. "Any online
bookings we haven't accepted?" means `list_from: "online"`. When looking up a
specific confirmation number, `la_get_reservation` already searches all four.

## Scanning honestly

List tools walk a bounded number of pages and say so when they stop early. If a
scan truncates, **narrow the date range and say what you covered** — never
present a partial list as a complete count. The operator is often asking
because a number matters for a decision.

## Quote conversion is a heuristic

`la_quote_conversion_report` matches quotes to reservations by passenger name
plus pickup date. That is genuinely useful and genuinely fallible: a trip
rebooked under a different name, a changed pickup date, or a group booking can
all break the match.

Present the results as **leads to check, never as verdicts**. "These eight look
unconverted — worth a call" is right. "We lost eight quotes" is not. Spot-check
anything surprising with `la_get_quote` and `la_get_reservation` before the
operator acts on it.

## Talking about money

- Totals are reservation **grand totals**. Farm-in/out costs and settlements are
  not netted out, so this is booked revenue, not profit.
- **Cancelled and no-show trips are reported separately** and excluded from
  booked totals. If an operator's number disagrees with yours, this is usually
  why — say which basis you used.
- `la_get_reservation` gives the full money picture for one trip: grand total,
  payments and deposits taken, and the balance still due.

## Presenting results

- **Lead with the answer.** "Six runs tomorrow, first pickup 5:40am" beats a
  table they have to scan.
- **Schedules read chronologically**, grouped by day, with pickup time,
  passenger, and vehicle. That's how a dispatcher thinks.
- **Use confirmation numbers.** They're how the operator finds the trip in their
  own system, so include them whenever you name a trip.
- **Flag the operationally interesting things** without being asked: an
  unassigned driver on a trip tomorrow, an unaccepted online booking, a large
  balance due on a job about to run.

## Making changes

Three things Otto can change, each in two steps:

- **Book a new reservation** — `la_prepare_reservation`. Needs the passenger,
  pickup date and time, pickup address, vehicle type and service type; drop-off,
  passenger count, phone, email, flat rate and trip notes are optional. Vehicle,
  service, driver and car names are matched against the operator's *own*
  LimoAnywhere dropdowns, so "Sprinter" or "business class sedan" both work; if
  a name doesn't match, the tool lists the real choices — pick with the operator
  rather than guessing.
- **Change an existing reservation** — `la_prepare_reservation_update`. Status
  (Assigned, Cancelled, Late Cancel, No Show, Done…), driver, car, pickup date
  or time, passengers, vehicle, service, flat rate, passenger phone/email, or
  the dispatch notes. Give only what changes. Cancelling a trip is a status
  change to Cancelled — Otto never deletes reservations.
- **Add a note** — `la_prepare_note`. Appends a trip note (optionally hidden
  from the customer) or replaces the driver-facing dispatch notes, without
  touching anything else on the trip.

The flow, every time:

1. Call the prepare tool. It reads LimoAnywhere, resolves every name to the
   operator's real options, and returns a **preview with a token**. Nothing has
   changed yet.
2. **Show the operator the preview** — what will be created or what goes from
   what to what — and ask if it's right. Do not paraphrase away details like
   the date, the rate, or which driver.
3. **Only after they explicitly say yes**, call `la_confirm_action` with the
   token. A "sure" to a different question, or silence, is not a yes. If they
   want anything different, prepare again; never edit a preview in your head
   and confirm the old token.
4. Report exactly what `la_confirm_action` says came back. It re-reads the
   reservation after saving, so if LimoAnywhere kept a different value it says
   so — pass that on rather than smoothing it over.

Tokens are single-use and expire after 15 minutes, so a stale approval can't
fire later. If a token is refused, prepare again and show the new preview.

What Otto deliberately does **not** do in LimoAnywhere: delete reservations,
take or record payments, convert quotes, accept online bookings, or send
confirmation emails and texts (a new reservation is saved with "Do Not Send",
and the operator sends the confirmation from LimoAnywhere when they're ready).
Say so plainly and point them to LimoAnywhere for those.

A new reservation is saved with the flat rate given (LimoAnywhere's automatic
fees and taxes are applied on top, exactly as its own form does), or with no
rate for the operator to price. Read the grand total back from the
confirmation message rather than quoting the flat rate as the price.

## When something isn't working

`la_check_connection` verifies the saved login works and that both quotes and
the calendar can be read. If the login is being rejected, LimoAnywhere is
usually the cause — a changed password, or a user whose permissions were
reduced. The fix is `/otto-setup` again with current credentials.

LimoAnywhere is an older system and does go down; `la_check_connection`
distinguishes "our login is wrong" from "LimoAnywhere is having trouble."
