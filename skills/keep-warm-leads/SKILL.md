---
name: keep-warm-leads
description: Keep warm LinkedIn leads from going cold. Plans the next few touches for one person (a follow-up chain on LinkedIn or email), records where each deal stands (agreed, booked, no-show, call done, proposal sent, trial, paid, closed), logs email, WhatsApp and call touches so nothing double-sends, and works the daily list of warm people whose next touch is due. Use when the user asks who to follow up with, what is due today, to follow up after a call, a no-show or a proposal, to chase someone who went quiet, to set or change a lead's stage, to save someone's email, to log an email or call, or to put warm follow-ups on a schedule.
---

# Keep warm leads warm

A campaign ends after its last message. Everything after that (a reply, a call, a proposal, a trial) has nothing scheduled, so it gets forgotten. This skill keeps those people moving. You write every word; PoliteReach holds the plan, the timing and the stop rules.

## First: what must be connected

1. **The PoliteReach connector** (`li_*` tools). Missing: use the `get-started` skill.
2. **The user's outreach skill**, for the voice and the stage templates (`references/stage-templates.md`). Missing: offer the `outreach-skill-builder` skill; until then, write in the thread's own tone and ask before every chain.
3. **An email connector, only for email steps** (Gmail, Outlook or similar, with a send or draft tool). Without one, email steps come to the user as text to send themselves. Never suggest one to a user who follows up on LinkedIn only.

## 1. Follow-up chains

A chain is the next 1-4 touches for ONE person from ONE account, inside or outside a campaign.

**When to offer one:**
- A warm thread went quiet and the next move is to wait on them.
- After a call (`call_done`), a no-show, a proposal, a trial start.
- After the booking follow-up closed with no booking, for a later check-in.
- Not while a campaign sequence still runs for them, and not instead of booking nudges: someone who agreed to a call gets `li_track_booking` (the `book-agreed-calls` skill).

**Writing the steps:**
- Start from the stage's template in the user's outreach skill (`references/stage-templates.md`). No template: propose 2-3 steps.
- Write each step for THIS person from the thread: their words, what was agreed, the next step. Every slot filled now; PoliteReach never writes text at send time.
- `delayDays` (1-60) = days after OUR previous message to them on any channel (LinkedIn or a logged touch), then after the previous step. Gaps strictly grow (for example 3, 7, 14). The last step is the close.
- Each step: `channel` (`linkedin` or `email`), `mode` (`send` or `draft`), `text`.
  - `send`: LinkedIn only. Goes out by itself when due, inside the account's sending hours. Never send it by hand.
  - `draft`: waits in the warm queue (`li_warm_due`) for a yes: the user's in chat, or a warm routine the user set to send mode (it sends a due LinkedIn draft step word for word, only while nothing new happened in the thread). Default for anything sensitive; tell the user a send-mode routine sends drafts too.
  - `email` is ALWAYS a draft: PoliteReach never sends email. When due, you send it through the user's email connector on their yes, then `li_log_touch` (`channel: "email"`, `direction: "out"`).
- A step completes by itself once any message of ours reaches them after it fell due (sent with `li_send_message` / `li_send_reply`, or an email logged with `li_log_touch`), so a step sent by hand is never sent again.

**Create it:**
1. `li_create_followup_chain` with `profileUrl`, `sessionLabel` (the account the thread is on), `steps`, a short `label`, and `stage` when it comes from a stage template. It previews each step's due time.
2. Show every step: text, channel, mode, due date. On the user's yes, call again with `confirmChain: true`.
3. One active chain per person per account. A second is refused; replace only on the user's yes (`replaceExisting: true` cancels the old one).

**It stops by itself** when they write on any channel (the LinkedIn thread is also read live right before each send), a call is booked, a final verdict is recorded (`li_mark_outcome` `won`, `not_interested` or `no_outcome`; `interested` does not stop it), or they go on the do-not-contact list. A send that fails ends the chain (`send_failed`); it is never retried. Read chains with `li_list_chains` (default `active`); stop one with `li_cancel_chain` (`chainId`, or `profileUrl` + `sessionLabel`) and a short `reason`.

## 2. Stages

Where each deal stands, separate from the campaign. `li_set_stage` with `profileUrls` and `stage`:

| Stage | Means |
|---|---|
| `agreed` | Said yes to a call, not booked yet |
| `booked` | A call is on the calendar |
| `no_show` | The call time passed and they did not join |
| `call_done` | The call happened. Pass `note` (what was agreed), `nextStep`, `owner`, `dueDate` (YYYY-MM-DD) |
| `proposal_sent` | A proposal or quote is out |
| `invited_not_activated` | Invited to sign up for the user's product, never got started. Only for products with sign-ups. Set automatically (`source` `auto`) on the PoliteReach team's own install |
| `trial` | On a trial |
| `paid` | Paying. Leaves the warm list |
| `closed` | Done either way. Leaves the warm list |

- `stage: null` clears it. `source: "routine"` when a scheduled task set it; default `manual`.
- A stage is not a verdict. Record the outcome with `li_mark_outcome` (`meeting_booked`, `won`…) where one applies, before any chain.
- Then offer that stage's chain from the user's outreach skill.

## 3. Other channels

- **An email address appears** (in a thread, a calendar booking, the user's mail): `li_set_contact_email` with `profileUrl` and `email`. Only an address that is clearly theirs.
- **Every touch off LinkedIn** (email, WhatsApp, call, other): `li_log_touch` with `profileUrl`, `channel`, `direction` (`in` = they reached the user, `out` = the user reached them), `at` (ISO, default now) and a short `note`.
- An **inbound** touch stops their chains, their campaign follow-ups and booking nudges: they are talking already. It is not a LinkedIn reply and records no verdict.
- An **outbound** touch counts as our last message: chain delays restart from it.

## 4. The daily warm queue

`li_warm_due` (all accounts, or `sessionLabel`), with `includeThread: true` when you will write. `limit` goes up to 100 and there is no paging. `truncated` or `scanTruncated` true: say the list is incomplete ("N more due, not shown", with `total`) and never call it complete. `unreadable`: an unreadable list is not an empty one, never say "nothing due".

Most overdue first. Work each row by its `reasons`:

- `chain_step_due`: `draftedStep` holds the step's text. `draftedStep.sendsItself` true: a `send` step past its time (sending hours or a hold); PoliteReach sends it, so never send it by hand, just say when. Otherwise re-read the thread; still right: show it for the user's yes, then send (LinkedIn: `li_send_message` with the row's account; email: the user's email connector, then `li_log_touch` out). Something changed: rewrite it, show both.
- `next_step_date`: the stage's next step (`stageNextStep`) is due today. Draft the touch that does it, or remind the user if it is not a message.
- `they_wrote_last`: their latest message on any channel is newer than ours. Draft an answer in the user's voice. A LinkedIn reply is also in `answer-replies`: follow that skill's calendar check.
- `quiet`: a live stage, no chain, nothing either way for 7 days (our policy). Propose the stage's chain.

Each row also carries `lastTouch` per channel, `stage` fields, `email`, `unreadStreak` and `suggestedChannel`.

- `unreadStreak` = our last messages on LinkedIn with no read receipt. LinkedIn receipts can be switched off, so it never means "unread": say "no read receipt on the last N messages".
- 2 or more with an email stored: `suggestedChannel` is `email`; suggest an email step. `emailMissing: true`: say "email (no address stored)" and ask whether the user has one.

## 5. Duplicate-send guard

`li_send_message`, `li_send_reply` and `li_send_bulk_messages` refuse the SAME text to the SAME person within 10 minutes. The refusal says when it went out or was queued.

- A refusal means it already went. Check the thread (`li_conversation_history`) or `li_job_result`; never resend.
- `force: true` only when the user explicitly wants the identical text sent twice. It skips this guard only, never the do-not-contact list, plan or holds.
- "Could not check for a duplicate": wait a minute and try once more; never `force` around it.

## Message rules

At most three short sentences, one idea per line. No em dashes, exclamation marks, emojis or filler ("just checking in", "circling back", "bumping this"). Each touch brings one new reason or one clear ask. Never re-pitch what the thread covered. Never invent a price, client, metric or date. Names: all capitals or all lowercase get normal capitals; mixed case stays as written.

## Stop conditions

A security check or sign-in error: stop and tell the user to reconnect the account. Never retry around it.

## Put it on a schedule

To work the warm queue every morning, follow `references/scheduled-task.md`.
