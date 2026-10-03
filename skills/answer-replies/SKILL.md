---
name: answer-replies
description: Find every LinkedIn reply waiting on the user, draft answers in their voice, send the approved ones through PoliteReach and record how each conversation went. Use when the user asks who replied, what needs an answer, to clear or triage their LinkedIn inbox, to answer replies, to record outcomes, or to put reply handling on a schedule.
---

# Answer LinkedIn replies

Work the whole queue in batches. Every tool here takes a list, so a queue is about six calls, not one browser session per person.

## 1. Find what is owed

Per account, or once with no account for all of them:

- `li_replies_to_answer` with `includeThread: true`: people in campaigns who replied, each with the end of their stored thread.
- `li_list_conversations` with `onlyNeedsReply: true`, `onlyAddressable: true`, `sinceDays: 30`, per account: catches people never imported into a campaign.

Merge and dedupe. Ignore threads between the user's own accounts and LinkedIn system messages.

Read the full thread before judging: `li_conversation_history`. Only if nothing is stored (an empty result means never read, not never wrote), `li_read_conversation` for that one person. Never one live read per person for the whole queue.

## 2. Decide, one per thread

- **Flag for the user, never send:** pricing beyond what their skill allows, contracts, NDAs, invoices, anything legal, anything senior or unclear, a request to use their own scheduler or to contact someone else. Give their last message, why, and a draft the user can send.
- **Reply:** you understand the ask and the answer needs no invented fact. Draft in the user's voice (their outreach skill). If their last message is over 14 days old, open with one short "sorry, this slipped past me."
- **They agreed to a call:** reply with the plain booking link (PoliteReach adds their name), then `li_track_booking` with `agreed: true`, their words as `agreedDayText`, and `agreedDate` (YYYY-MM-DD) if they named a day. A row flagged `bookingPending` wrote again after agreeing: answer what they said, never a booking nudge.
- **Skip:** dead threads (a flat no, a thumbs up, a vendor pitching the user). Unsure between reply and skip: flag.

## 3. Send only what the user approved

Show every draft. On a yes: one `li_send_reply` with `replies: [{profileUrl, message}, ...]` and `confirmSend: true`. Each reply goes from the account the conversation is on. A `jobId` means queued, not failed. Never send twice to the same person; check with one `li_job_result` call (no jobId).

## 4. Record a verdict for every thread read

One `li_mark_outcome` per verdict, with `profileUrls` and a short `note`:

- `not_interested`: a clear no.
- `interested`: wants to talk, nothing agreed yet.
- `meeting_booked`: a call is actually on the calendar. Never `won`.
- `won`: a deal actually agreed. A demo request is not won.
- `no_outcome`: ran its course with nothing either way.

Other marks:
- Still open after you answered: `li_mark_reply_handled`. It records that the user did their part, not how it went.
- "Try again in (month)": `li_snooze_reply` with `until`. Nothing is sent when it expires.
- A thread not worth a verdict on the person: `li_mark_reviewed` with `conversationUrn`, `lastMessageAt` (the row's `lastActivityAt`), `decision` (e.g. `skip`, `vendor_pitch`, `no_action`) and `sessionLabel`.
- Flagged threads: mark nothing.

A marked conversation reopens by itself if they write again.

## Message rules

At most three short sentences, one idea per line. No em dashes, exclamation marks, emojis or filler ("just checking in", "circling back"). End on a concrete ask. Never invent a price, client, metric or date.

## Stop conditions

A security check or sign-in error: stop and tell the user to reconnect the account. Never retry around it.

## Put it on a schedule

To run this unattended, follow `references/scheduled-task.md`.
