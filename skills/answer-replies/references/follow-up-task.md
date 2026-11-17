# Next touches for an open thread

For one person whose thread is open and waiting on them: we answered them, sent what they asked for, a call happened, a proposal went out. Book the next touches as a follow-up chain: PoliteReach holds the text and the timing, and stops the chain by itself the moment they write (on any channel), agree to a call, book one, get a final verdict (`won`, `not_interested`, `no_outcome`) or go on the do-not-contact list. No scheduled task is needed.

The full rules (stages, email steps, the warm queue) are in the `keep-warm-leads` skill. This is the short form.

## One owner per open thread

- They agreed to a call: the booking follow-up owns them, from `li_track_booking` until they book or it closes them. It alone nudges and closes them.
- Any other open thread where we spoke last and wait on them: a follow-up chain.
- They never replied: their campaign sequence (it stops when they reply).
- "Not now, try in (month)": `li_snooze_reply` with `until`, no chain.
- A final verdict, do-not-contact, or a thread flagged for the user: nobody.

`li_track_booking` ends every active chain on the person (any account of the team) in the same write and says so (`chainsCanceled`, `chainNote`); `li_mark_booked` (a new booking) and `li_snooze_reply` (with a date) do the same. `li_cancel_chain` first is not needed. `li_create_followup_chain` refuses anyone who agreed to a call, and anyone with a call booked unless the chain is for an after-call stage (`no_show`, `call_done`, `proposal_sent`, `invited_not_activated`, `trial`). Someone only sent the booking link, who never agreed, is allowed: the booking follow-up never chases them, so the chain is their only follow-up.

## When to start one

After every reply of ours that leaves the thread open: we answered a question, sent what they asked for, or pushed the call (the booking link or "want to get on a call?") and they have not agreed. Never when they agreed, booked, got a final verdict, were snoozed or the thread was flagged.

## Part 1: settle it with the user

1. Is it already covered? Their `li_replies_to_answer` row carries `activeChain` (null = none); otherwise `li_list_chains` with their `profileUrl`. `activeChainUnreadable` on a row, or `activeChainLookupFailed` on `li_list_conversations`, means the read failed: use `li_list_chains`. An active chain: replace it (`replaceExisting: true` cancels the old one) only when our new reply changes what it should say. `replaceExisting` reaches only the chain on the same account: one on another account (its `accountId` differs) still owns the thread, so leave it, or end it with `li_cancel_chain` first when the new chain should take over. `endsAtNextPass: true` means they wrote after it started, so it ends by itself within the hour: after you answer, replace it.
2. The steps. The user's outreach skill decides (`references/stage-templates.md`: its "Follow-up after our reply" section, or the stage's template). Where it says nothing, use these defaults. `delayDays` counts from our previous message, then from the previous step, so each number below is a GAP; the day after our reply it lands on is in brackets:

   | Situation | Steps (days) | Mode |
   |---|---|---|
   | We answered a question or pushed the call, no agreement | 3 [day 3]: did that answer it, plus the one concrete next step (if the link is in the thread, point back to it, never paste it again). 4 [day 7]: one new useful thing not yet in the thread (a proof point or product fact), plus the ask. 7 [day 14]: the close | send |
   | We sent what they asked for (a demo video, a case study, information) | 2 [day 2]: did you get a chance to look. 4 [day 6]: ask for the call. 7 [day 13]: the close | send |
   | They said "will do", "let me check with the team" or "I'll get back to you" | 5 [day 5]: a light check. 7 [day 12]: the close | send |
   | `no_show` | 1 [day 1]: offer to rebook. 4 [day 5]: the close | send |
   | `call_done` [1, 5, 12], `proposal_sent` [3, 7, 14] | the stage template | draft |
   | `trial`, `invited_not_activated` [2, 5, 10] | the stage template | send |

   The close: "Should I close this off? If it's not a fit right now, mind sharing what's holding you back?"
   - `delayDays` (1-60): gaps strictly grow; the last step is the close; 1-4 steps. A step is never due less than 2 hours after the chain is created. Due times follow our latest message on any channel, so a chain created right after a reply is queued counts from that reply once it goes out.
   - `channel`: `linkedin`, or `email` when the thread moved to email (needs their address: `li_set_contact_email`).
   - `mode`:
     - Send (LinkedIn only): the exact text approved now goes out by itself when due, inside the account's sending hours.
     - Draft: when it falls due it waits in `li_warm_due` for the user's yes. Email steps are always drafts.
3. The text of every step, written now for this person in the user's voice: what was said, what was agreed, the next step, never what the thread already covered. Nothing is written later; PoliteReach never generates text at send time.

## Part 2: create the chain

1. `li_create_followup_chain` with `profileUrl`, `sessionLabel` (the account the thread is on), `steps` (`delayDays`, `channel`, `text`, `mode`), a short `label` (for example "after our reply" or "after proposal") and `stage` when it came from a stage template. Without `confirmChain` it is a dry run that previews each step's due time. Pass an `idempotencyKey` and reuse it on a retry.
2. Show every step (text, channel, mode, due date), next to the reply it follows. On the user's yes (one yes covers a reply and its chain), the same call with `confirmChain: true`. A refusal names who owns them (the booking follow-up, a running chain): relay it, never work around it.
3. Report each chain on one line: name, account, each step's due date. Say it stops by itself if they write first.

To stop it later: `li_cancel_chain` with `chainId`, or `profileUrl` + `sessionLabel`, and a short `reason`.

## Rules for the steps

Max 3 short sentences, no em dashes, no exclamation marks, no emojis, no "just checking in" or "circling back", never re-pitch what the thread already covered, never invent a fact. Someone who agrees to a call gets `li_track_booking`: it ends their chain and the booking follow-up takes over. If a call was agreed anywhere in the thread, search the user's calendar for them first (if connected): booked = no chain, `li_mark_booked`.
