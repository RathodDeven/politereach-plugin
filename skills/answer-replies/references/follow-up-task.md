# Next touches for a warm thread

For one person whose thread has nothing scheduled: they replied, a call happened, a proposal went out, or you are waiting on them. Book the next touches as a follow-up chain: PoliteReach holds the text and the timing, and stops the chain by itself the moment they write (on any channel), book a call, get a verdict or go on the do-not-contact list. No scheduled task is needed.

The full rules (stages, email steps, the warm queue) are in the `keep-warm-leads` skill. This is the short form.

## Part 1: settle it with the user

1. Is it already covered? `li_list_chains` with their `profileUrl`. An active chain on that account: change it only on the user's yes (`replaceExisting: true` cancels the old one). A campaign sequence still running for them, or a booking follow-up (they are in `li_bookings` with `status: "agreed"`): leave it to that.
2. The steps. Start from the stage's template in the user's outreach skill (`references/stage-templates.md`) when the person has a stage; otherwise one step about 7 days after our last message, unless the user names a date, plus a close if they want one. 1-4 steps.
   - `delayDays` (1-60): days after our previous message to them (LinkedIn or a logged email or call), then after the previous step. Gaps strictly grow; the last step is the close.
   - `channel`: `linkedin`, or `email` when the thread moved to email (needs their address: `li_set_contact_email`).
   - `mode` [draft]:
     - Draft: when it falls due it waits in `li_warm_due` for the user's yes. Email steps are always drafts.
     - Send (LinkedIn only): the exact text approved now goes out by itself when due, inside the account's sending hours.
3. The text of every step, written now for this person in the user's voice: what was said, what was agreed, the next step. Nothing is written later; PoliteReach never generates text at send time.

## Part 2: create the chain

1. `li_create_followup_chain` with `profileUrl`, `sessionLabel` (the account the thread is on), `steps` (`delayDays`, `channel`, `text`, `mode`), a short `label` (for example "after proposal") and `stage` when it came from a stage template. The reply previews each step's due time.
2. Show every step (text, channel, mode, due date). On the user's yes, the same call with `confirmChain: true`.
3. Tell the user when the first step is due, and that it stops by itself if they write first.

To stop it later: `li_cancel_chain` with `chainId`, or `profileUrl` + `sessionLabel`, and a short `reason`.

## Rules for the steps

Max 3 short sentences, no em dashes, no exclamation marks, no emojis, no "just checking in" or "circling back", never re-pitch what the thread already covered, never invent a fact. Someone who agreed to a call gets `li_track_booking` and the booking nudges, never a chain on top. If a call was agreed anywhere in the thread, search the user's calendar for them first (if connected): booked = no chain, `li_mark_booked`.
