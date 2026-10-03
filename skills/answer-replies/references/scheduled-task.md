# Scheduled task: LinkedIn replies

Create nothing until the user has answered part 1.

## Part 1: ask first

Ask in one message, numbered, defaults in brackets, so the user can answer "defaults". If their outreach skill already answers one, say what you took from it. Ask their timezone if unknown.

1. Which PoliteReach accounts should it cover? Use the names from `li_list_accounts`. [all of them]
2. Send or draft? Send: it answers the clear replies itself and flags the rest. Draft: it writes every reply and sends nothing. [draft]
3. What must always come to you instead of being answered? [pricing beyond what your skill allows, contracts, NDAs, invoices, anything legal]
4. Your booking link, if your skill does not have it.
5. When should it run, in your timezone? [every 2 hours from 10:00 to 20:00, every day]

## Part 2: create the task

Create a recurring scheduled task named "LinkedIn replies" at the chosen times, in the user's timezone, with the TASK PROMPT below: CONFIG filled in, the rest word for word. Attach the user's outreach skill or name it in CONFIG.

Then tell the user the task name and next run, and remind them to set every tool it uses to always allow: nobody is there to approve a tool while it runs. Choosing send mode here is the user's standing approval to send. Offer one run now.

## TASK PROMPT

```text
Answer my LinkedIn replies via PoliteReach. Nobody is watching this run: never ask a question, decide and act within MODE.

CONFIG
- Skill: (my outreach skill name)
- Accounts: (my account names)
- MODE: (send | draft). In draft mode never call li_send_reply: put each reply you would have sent under DRAFTS in the report.
- Always flag, never answer: (my list)
- Booking link: (my link)

SETUP
1. Load my outreach skill (named in CONFIG) and follow its voice and rules for every word.
2. Make sure the PoliteReach tools (li_*) are loaded. If they are missing, notify me "(task name) failed: PoliteReach not connected" and stop.
3. li_list_accounts. Use the CONFIG accounts; if a name no longer matches, pick the closest. Skip and report any account that is paused or needs signing in again (fix: Reconnect on the PoliteReach Accounts page).

FIND WHAT IS OWED
4. Per account: li_replies_to_answer with includeThread true, AND li_list_conversations with onlyNeedsReply true, onlyAddressable true, sinceDays 30. People never imported into a campaign only show up in the second.
5. Merge and dedupe. Ignore threads between my own accounts and LinkedIn system messages. Only threads whose last message is 30 days old or newer.
6. Read each thread in full first: li_conversation_history, or li_read_conversation on that account if nothing is stored.

DECIDE, one per thread
A. FLAG, never send: anything on the always-flag list, someone asking us to use their own scheduler or contact someone else, anything senior or unclear. Give their last message, why, and a draft I can send myself.
B. SEND when confident: you understand the ask and the answer needs no fact you would have to invent. li_send_reply on the account the thread is on, confirmSend true, batch with replies[]. A job id means queued; never send twice to anyone.
If they agreed to a call: send my plain booking link (PoliteReach adds their name), then li_track_booking with agreed true, their words as agreedDayText and agreedDate if they named a day. A row flagged bookingPending wrote again after agreeing: answer what they said, never a booking nudge.
If their last message is over 14 days old, open with one short "sorry, this slipped past me."
C. SKIP dead threads (a flat no, a thumbs up, a vendor pitching us). When unsure between send and skip, flag.

RECORD A VERDICT for every thread read, one li_mark_outcome call per verdict per account with a short note: not_interested (a clear no), interested (wants to talk, nothing agreed), meeting_booked (a call actually booked), won (a deal actually agreed; a demo is not), no_outcome (ran its course). For someone in no campaign, pass their profile URL, conversationUrn and sessionLabel. Still open after a send: li_mark_reply_handled. "Try again in (month)": li_snooze_reply. Skipped dead threads: li_mark_reviewed with conversationUrn, lastMessageAt, decision (skip | vendor_pitch | no_action) and sessionLabel, so the next run does not re-read them. Flagged: mark nothing.

MESSAGE RULES (hard)
Max 3 short sentences, one idea per line, blank line between lines. No em dashes, no exclamation marks, no emojis. No filler: no "just checking in", "circling back", "hope this finds you well". Match the tone of the thread and my skill's voice. End on a concrete ask, never "let me know". Never invent a price, client, metric or date.

If any tool returns a security-check or sign-in error, stop work on that account and report it; never retry around it.

REPORT: FLAGGED first, then SENT (or DRAFTS), each message in a code block, then one "Skipped: N" line per account. Nothing at all: "Nothing owed."
Notify me once only when something was sent, drafted or flagged, first line the counts. Notify too if the run could not work (connector failing, every account blocked).
```
