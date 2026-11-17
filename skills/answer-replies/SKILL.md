---
name: answer-replies
description: Find every LinkedIn reply waiting on the user, draft answers in their voice, send the approved ones through PoliteReach, start the follow-ups for every thread left open, and record how each conversation went. Use when the user asks who replied, what needs an answer, to clear or triage their LinkedIn inbox, to answer replies, to record outcomes, or to put reply handling on a schedule.
---

# Answer LinkedIn replies

**Connect the calendar first.** Ask the user to connect their calendar in this app's connector settings (Google Calendar or Microsoft 365 / Outlook) if it is not: it is how you see who already booked a call. Without it, anyone with their booking link is flagged for them, never answered.

Work the whole queue in batches. Every tool here takes a list, so a queue is about six calls, not one browser session per person.

## 1. Find what is owed

- `li_replies_to_answer` with `includeThread: true`, once with no account (covers all accounts): people in campaigns who replied, each with the end of their stored thread and its `activeChain` (null = none). Read `truncated`; raise `limit` until it is false.
- `li_list_conversations` with `onlyNeedsReply: true`, `onlyAddressable: true`, `sinceDays: 30`: catches people never imported into a campaign. A live browser read, one account per call (no account = the default account only): loop over the accounts from `li_list_accounts`. `reachedEnd` false = partial: say so, never "nothing owed".

Merge and dedupe. Ignore threads between the user's own accounts and LinkedIn system messages.

Read the full thread before judging: `li_conversation_history`. Only if nothing is stored (an empty result means never read, not never wrote), `li_read_conversation` for that one person, with `profileUrl` and `sessionLabel` so the read is stored for next time. Never one live read per person for the whole queue.

## 2. Decide, one per thread

- **Flag for the user, never send:** pricing beyond what their skill allows (a price question their skill gives a line for is a normal reply), contracts, NDAs, invoices, anything legal, anything senior or unclear, a request to use their own scheduler or to contact someone else. Give their last message, why, and a draft the user can send.
- **Reply:** you understand the ask and the answer needs no invented fact. Draft in the user's voice (their outreach skill). If their last message is over 14 days old, open with one short "sorry, this slipped past me."
- **Calendar check first (hard):** a row with `bookingCheckRequired` (our booking link is out, or a call is booked), anyone who agreed to a call or says they booked: search the user's calendar for them (full name, then first name + company) BEFORE writing. People book within minutes of saying yes; PoliteReach reads no calendar.
  - Booked: send NOTHING (no "grab a slot", no "see you then"; the booking invite already confirmed it). `li_mark_booked` with the event id and start; unclear match → `li_match_contacts`, mark only on `match`. No `li_track_booking`.
  - Not booked: answer, and pass `calendarChecked: true` to `li_send_reply` (refused without it).
  - No calendar connector: never send to them. Flag for the user.
- **They agreed to a call (not booked):** reply with the plain booking link (PoliteReach adds their name); if our link is in the last few messages, point to it instead of pasting it again. Then `li_track_booking` with `agreed: true`, their words as `agreedDayText`, and `agreedDate` (YYYY-MM-DD) if they named a day. A row flagged `bookingPending` wrote again after agreeing: answer what they said, never a booking nudge. `li_track_booking` already records them as `interested`: give them no other verdict and no `li_mark_reply_handled`. It also ends any follow-up chain on them (`chainsCanceled`): the booking follow-up owns them now, so no chain.
- **They say they booked, but the calendar shows nothing yet:** send nothing; `li_mark_outcome` `meeting_booked` with a note. The booking check attaches the event later.
- **Skip:** dead threads (a flat no, a thumbs up, a vendor pitching the user). Unsure between reply and skip: flag.

## 3. Send only what the user approved

Before drafting, check the sending account's `plan` in `li_list_accounts`: a lapsed or ending trial may refuse the send.

Show every draft, each with the follow-up chain that comes after it (below). On a yes, search the calendar once more for anyone with `bookingCheckRequired` and drop whoever booked meanwhile. Then one `li_send_reply` with `replies: [{profileUrl, message}, ...]`, `confirmSend: true`, and `calendarChecked: true` when the batch holds a checked booking row. Each reply goes from the account the conversation is on. A `jobId` means queued, not failed. Never send twice to the same person; check with one `li_job_result` call (no jobId). A refusal saying the identical text already went to them in the last 10 minutes means it was sent: check the thread, never resend. `force: true` only when the user explicitly wants the same text twice. Then start the chains (below).

## 4. Record a verdict for every thread read

One `li_mark_outcome` per verdict, with `profileUrls` and a short `note`:

- `not_interested`: a clear no.
- `interested`: wants to talk, nothing agreed yet.
- `meeting_booked`: a call is actually on the calendar. Never `won`.
- `won`: a deal actually agreed. A demo request is not won.
- `no_outcome`: ran its course with nothing either way.

A row's `suggestedOutcome` (`not_interested` from a clear opt-out) is a suggestion only, never written: read their message, and only if it really is a no, `li_mark_outcome` `not_interested`. "Remove me" may also warrant `li_suppress_profiles` (on the user's yes).

Then work the verdict queue: `li_marked_conversations` with `outcome: "undecided"` and `dueOnly: true` lists people who replied 48+ hours ago with no verdict (the older part of `repliedNoVerdict` in `li_outcome_summary`, so its count is smaller). Record a verdict for each; `owedReply` rows still need an answer first.

Other marks:
- Still open after you answered: `li_mark_reply_handled`. It records that the user did their part, not how it went. The thread also gets a chain (below).
- "Try again in (month)": `li_snooze_reply` with `until`, and no chain: a dated snooze ends their chains. Nothing is sent when it expires.
- A thread not worth a verdict on the person: `li_mark_reviewed` with `conversationUrn`, `lastMessageAt` (the row's `lastActivityAt`), `decision` (e.g. `skip`, `vendor_pitch`, `no_action`) and `sessionLabel`.
- Flagged threads: mark nothing.
- Where the deal now stands (a call done, a proposal out, a trial started): `li_set_stage` with `profileUrls` and the stage; `call_done` takes `note`, `nextStep`, `owner` and `dueDate`. Then offer that stage's chain.
- The conversation moved to email, WhatsApp or a call: `li_set_contact_email` when they give an address, and `li_log_touch` for each touch off LinkedIn (`direction: "in"` for theirs stops their chains and campaign follow-ups).

A marked conversation reopens by itself if they write again.

## After you reply: one owner per open thread

Every open thread has exactly one owner that follows up, so nobody is chased twice or forgotten:

- Agreed to a call: the booking follow-up (`book-agreed-calls`), from `li_track_booking` until they book or it closes them.
- Any other open thread where we spoke last and wait on them: a follow-up chain.
- Never replied: their campaign sequence.
- "Not now, try in (month)": `li_snooze_reply`, no chain.
- A final verdict, do-not-contact, or flagged for the user: nobody.

Start a chain after every reply that leaves the thread open: you answered a question, sent what they asked for, or pushed the call (the link or "want to get on a call?") and they have not agreed. None when they agreed, booked, got a final verdict, were snoozed or were flagged. Check the row's `activeChain` first: replace it (`replaceExisting: true`) only when your reply changes what it should say, or when `endsAtNextPass` is true (they wrote since it started). A chain on another account (its `accountId` differs) is not replaced that way: leave it, or end it with `li_cancel_chain` first. The default shapes (which the user's outreach skill can override) and the steps are in `references/follow-up-task.md`; the same applies after a call, a no-show or a proposal. Show each reply with its chain: one yes covers both, so send, then `li_create_followup_chain` with `confirmChain: true`.

If they agree to a call later, `li_track_booking` ends the chain itself (`li_cancel_chain` first is harmless, not needed). `li_create_followup_chain` refuses anyone who agreed, and anyone booked unless the chain is for an after-call stage. Report each chain on one line: name, account, step due dates. The whole warm-lead workflow is the `keep-warm-leads` skill.

## Message rules

At most three short sentences, one idea per line. No em dashes, exclamation marks, emojis or filler ("just checking in", "circling back"). End on a concrete ask. Never invent a price, client, metric or date. A name in all capitals or all lowercase gets normal capitals ("SOHAN" → "Sohan"); a mixed-case name ("McDonald", "deVries") stays exactly as written.

## Stop conditions

A security check or sign-in error: stop and tell the user to reconnect the account. Never retry around it.

## Put it on a schedule

To run this unattended, follow `references/scheduled-task.md`.
