---
name: pipeline-check
description: Answer where the user's LinkedIn outreach stands in PoliteReach. Use when the user asks how today went, how campaigns are doing, who accepted and what goes out to them next, how many invites are pending, how results or outcomes look, or wants someone added to their do-not-contact list.
---

# Where things stand

All reads below are instant and change nothing unless a step says otherwise. Reads without an account cover every account; scope with `sessionLabel` when the user means one.

## How did today go?

`li_daily_digest` per account: invites today and this week, acceptances, messages out, replies in, who is waiting, pending backlog, failures. A blocked account leads the report.

## How are my campaigns doing?

`li_list_campaigns`, then per campaign `li_campaign_contacts` with `view: "summary"` (counts over every contact) or `li_campaign_status`. Show invited, accepted, mid-sequence and stopped. `li_campaign_contacts` rows are paged: read `truncated` before calling a page the whole campaign.

## Who accepted, and what goes out next?

`li_accepted_pending_message`, then `li_contact_dossier` with `profileUrls` for the exact wording and due time. If something no longer fits:
- Change the words: `li_edit_step` (reversible).
- Hold one message: `li_cancel_steps` (cannot be undone).
- Stop the person entirely: `li_mark_stopped` (ends their sequence).
Each only on the user's yes.

## How many invites am I waiting on?

`li_pending_invites` per account: count and age. Read `reach`: `partial` or `failed` is not an empty list. Withdraw nothing unless the user asks: a withdrawn invite blocks re-inviting that person for about three weeks. On a clear yes: `li_withdraw_invites` with `confirmWithdraw: true`, then `li_withdraw_status`.

## How are results?

- `li_outcome_summary`: the human verdicts (won, meeting booked, interested, not interested, no outcome). `undecided` means nobody has judged those yet, not that they went badly. Suggest the `answer-replies` skill to close them out.
- `li_outreach_analytics` compares acceptance and reply rates by strategy, ICP or campaign. If it is refused for the user's access, use `li_campaign_status` per campaign.
- Invite pace: from `li_list_accounts` (`invitePace`). It is PoliteReach's own per-tier safety policy, not a LinkedIn rule, and cannot be raised. Never quote one flat number for everyone.

## Never contact these people

`li_suppress_profiles` with the `profileUrls` and a `reason`: preview with `confirmSuppress: false`, then `confirmSuppress: true`. Listed people are refused at import, before every invite and before every automated message. Read the list with `li_list_suppressed`; remove one with `li_unsuppress_profile`.

## What is the account doing right now?

`li_account_activity`: what runs now, what is queued, and why anything waits. "Queued" usually means another job is ahead on a shared browser. For setup or sign-in problems, use the `get-started` skill.
