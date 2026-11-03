# Scheduled task: LinkedIn replies

Create nothing until the user has answered part 1.

The task writes with the user's outreach skill, named "(company)-politereach-skill". Find it among their skills first. If theirs has another name, ask which one it is and use that name. If they have none, offer to build it first (the `outreach-skill-builder` skill, or `li_setup_guide` step `outreach_skill`) and stop.

## Part 1: ask first

Ask in one message, numbered, defaults in brackets, so the user can answer "defaults". If their outreach skill already answers one, say what you took from it. Ask their timezone if unknown.

1. Which PoliteReach accounts should it cover? Use the names from `li_list_accounts`. [all of them]
2. Send or draft? Send: it answers the clear replies itself and flags the rest. Draft: it writes every reply and sends nothing. [draft]
3. What must always come to you instead of being answered? [pricing beyond what your skill allows, contracts, NDAs, invoices, anything legal]
4. Your booking link, if your skill does not have it. And which calendar the bookings land on (connect it in this app: Google Calendar or Microsoft 365 / Outlook, with its tools allowed to run on their own), so it can see who already booked and never asks someone to book a call they booked minutes ago. Without one, anyone who has the link is flagged for you instead of answered.
5. When should it run, in your timezone? [every 2 hours from 10:00 to 20:00, every day]

## Part 2: create the task

Create a recurring scheduled task named "LinkedIn replies" at the chosen times, in the user's timezone, with the TASK PROMPT below: CONFIG filled in, the rest word for word. Attach the user's (company)-politereach-skill to the task and put its exact name in CONFIG, so every run loads it.

Then tell the user the task name and next run, and remind them to set every tool it uses to always allow: nobody is there to approve a tool while it runs. Choosing send mode here is the user's standing approval to send. Offer one run now so they can watch it.

## TASK PROMPT

```text
Answer my LinkedIn replies via PoliteReach. Nobody is watching this run: never ask a question, decide and act within MODE.

CONFIG
- Skill: (my company)-politereach-skill
- Accounts: (my account names)
- MODE: (send | draft). In draft mode never call li_send_reply: put each reply you would have sent under DRAFTS in the report.
- Always flag, never answer: (my list)
- Booking link: (my link)
- Calendar connector: (Google Calendar | Microsoft 365 | none)

SETUP
1. Load my outreach skill (named in CONFIG) and follow its voice and rules for every word. If it cannot be loaded, notify me "(task name) failed: skill not found" and stop.
2. Make sure the PoliteReach tools (li_*) are loaded (if this app defers tools, search for them first). If they are missing, notify me "(task name) failed: PoliteReach not connected" and stop.
3. li_list_accounts. Use the CONFIG accounts; if a name no longer matches, pick the closest. Skip and report any account that is paused or needs signing in again (fix: Reconnect on the PoliteReach Accounts page).

FIND WHAT IS OWED
4. Per account: li_replies_to_answer with includeThread true and threadMessages 12, AND li_list_conversations with onlyNeedsReply true, onlyAddressable true, sinceDays 30, limit 30. People never imported into a campaign only show up in the second.
5. Merge and dedupe. Ignore threads between my own accounts and LinkedIn system messages. Only threads whose last message is 30 days old or newer; leave older ones out of the report entirely.
6. Read each thread in full first: li_conversation_history, or li_read_conversation with the profileUrl and that account's sessionLabel if nothing is stored.

DECIDE, one per thread
A. FLAG, never send: anything on the always-flag list (a price question my skill gives a line for is a normal B), someone asking us to use their own scheduler or contact someone else, anything senior or unclear. Give their last message, why, and a draft I can send myself. Leave the thread in my queue with li_mark_replied and a note starting "ACTION NEEDED:" and the reason (someone in no campaign: pass their profile URL, conversationUrn, participantName and sessionLabel).
B. SEND when confident: you understand the ask and the answer needs no fact you would have to invent. li_send_reply on the account the thread is on, confirmSend true, batch with replies[]. A job id means queued; never send twice to anyone.
CALENDAR CHECK BEFORE ANY REPLY (hard): for every row with bookingCheckRequired, anyone who agrees to a call or says they booked, and any thread with my booking link in it, search my calendar for them (full name, then first name + company) before writing. Booked = an event for them created after the link went out. If booked: send NOTHING, li_mark_booked with the event id and start (unclear: li_match_contacts, mark only on match), no li_track_booking, and list them under BOOKED in the report. Search again right before li_send_reply and drop anyone who booked meanwhile. Not booked: send with calendarChecked true. No calendar connector: never send to them; FLAG them instead.
If they agreed to a call and have not booked: send my plain booking link (point to it if it is already in the last few messages) (PoliteReach adds their name), then li_track_booking with agreed true, their words as agreedDayText and agreedDate if they named a day. A row flagged bookingPending wrote again after agreeing: answer what they said, never a booking nudge. li_track_booking already sets them interested: give them no other verdict and no li_mark_reply_handled. They say they booked but no event shows yet: send nothing, li_mark_outcome meeting_booked with a note; the booking routine attaches the event later.
If their last message is over 14 days old, open with one short "sorry, this slipped past me."
C. SKIP dead threads (a flat no, a thumbs up, a vendor pitching us). When unsure between send and skip, flag.

RECORD A VERDICT for every thread read, one li_mark_outcome call per verdict per account with a short note: not_interested (a clear no), interested (wants to talk, nothing agreed), meeting_booked (a call actually booked), won (a deal actually agreed; a demo is not), no_outcome (ran its course). For someone in no campaign, pass their profile URL, conversationUrn and sessionLabel. Still open after a send: li_mark_reply_handled. "Try again in (month)": li_snooze_reply. Skipped dead threads: li_mark_reviewed with conversationUrn, lastMessageAt, decision (skip | vendor_pitch | no_action) and sessionLabel, so the next run does not re-read them. Flagged: no verdict (only the li_mark_replied note above).
A row's suggestedOutcome is a suggestion only, never a verdict: read their message, and mark not_interested only if it really is a no. "Remove me" or "stop messaging": list them under FLAGGED for a do-not-contact decision; never call li_suppress_profiles yourself.
VERDICT QUEUE: then li_marked_conversations with outcome "undecided" and dueOnly true (people who replied 48+ hours ago with no verdict). Read each thread (li_conversation_history) and record a verdict as above (none for a thread you flagged in this run: it waits on me); suggestedOutcome on these rows is a suggestion too. Rows listed under excluded.owedReply still need an answer: they are in the queue above.

MESSAGE RULES (hard)
Max 3 short sentences, one idea per line, blank line between lines. No em dashes, no exclamation marks, no emojis. No filler: no "just checking in", "circling back", "hope this finds you well". Match the tone of the thread and my skill's voice. Never repeat what the thread already covered. End on a concrete ask, never "let me know". Never invent a price, client, metric or date. A name in all capitals or all lowercase gets normal capitals ("SOHAN" → "Sohan"); a mixed-case name ("McDonald", "deVries") stays exactly as written.

If any tool returns a security-check or sign-in error, stop work on that account and report it; never retry around it.

REPORT: FLAGGED first, then BOOKED (no reply sent), then SENT (or DRAFTS), each message in a code block, then one "Skipped: N" line per account, then one line "Verdicts: N not interested, N meeting booked, N won, N no outcome, N left open, N booking tracked".
Notify me once only when something was sent, drafted or flagged: first line the counts, then one short line per flagged person saying why, so I can act from my phone. Notify too if the run could not work (connector failing, every account blocked).
EMPTY RUN = SILENCE: nothing sent, drafted, flagged or recorded means no notification and no message. End with exactly "Nothing owed."
```
