---
name: pipeline-check
description: Answer where the user's LinkedIn outreach stands in PoliteReach. Use when the user asks how today went, how campaigns are doing, which of a list of people are already in which campaigns, who accepted and what goes out to them next, how many invites are pending, how results, outcomes or an A/B test look, or wants someone added to their do-not-contact list.
---

# Where things stand

All reads below change nothing unless a step says otherwise. Most are instant; `li_pending_invites` opens the browser. Reads without an account cover every account; scope with `sessionLabel` when the user means one.

Ask for the compact form first and widen only when needed: filters and `fields` on `li_list_campaigns`, `view: "summary"` or the default compact rows on `li_campaign_contacts`, `format: "csv"` for a long table. Independent reads (several campaigns, several accounts) can run in parallel.

If a tool says PoliteReach could not reach its database, retry once, then report that account as unreadable, never as empty.

## How did today go?

`li_daily_digest` per account: invites today and this week, acceptances, messages out, replies in, who is waiting, pending backlog, failures. A blocked account leads the report.

## How are my campaigns doing?

`li_list_campaigns`: narrow with `namePrefix`, `sessionLabel` or `minContacts`; read `truncated`, and while it is true call again with `offset` = `nextOffset`. A campaign not launched yet sends nothing. Then per campaign `li_campaign_contacts` with `view: "summary"` (counts over every contact) or `li_campaign_status`. Report the `byConnectionStatus` and `byConversationState` counts as they are; there is no separate stopped count.

`li_campaign_contacts` rows are paged: read `truncated` before calling a page the whole campaign. Rows are compact by default (`stepCounts`, no message text); pass `includeSteps: true` only to read the copy or get step ids. To list only some people, filter on the server: `connectionStatus` (e.g. `["accepted"]`), `conversationState` (e.g. `["needs_reply"]`) or `hasInbound: true`; `matchingContacts` is the count.

## Which of these people are already in a campaign?

`li_check_targets` with the `profileUrls` (up to 1000) and `view: "people"`: one row per person listing every campaign they are in, on every account the user can see (connection, message, outcome, inbound), each campaign named once in `campaigns`. Anyone not in the table is in none. When the user is building new campaigns, pass `campaignNamePrefix` (or `campaignId`): anyone already in one of them is named in `batch.members`. The default `summary` view shows only each person's furthest bucket plus `otherCampaigns`.

## Who accepted, and what goes out next?

`li_accepted_pending_message`: rows already carry `firstMessageText` and `firstStepScheduledAt`. Flag rows whose `firstStepScheduledAt` is in the past, and any with `sendFailed`. If something no longer fits:
- Change the words: `li_edit_step` (reversible).
- Hold one message: `li_cancel_steps` (cannot be undone).
- Stop the person entirely: `li_mark_stopped` (ends their sequence).
Each only on the user's yes.

## How many invites am I waiting on?

`li_pending_invites` per account opens the browser and can take minutes; pass `maxInvites` for a quick look. Use `pendingOnLinkedIn` for the count. If `ageDays` is 0 on every row, report no ages. Read `reach`: `partial` or `failed` is not an empty list. Withdraw nothing unless the user asks: a withdrawn invite blocks re-inviting that person for about three weeks. On a clear yes: `li_withdraw_invites` with `confirmWithdraw: true`, then `li_withdraw_status`.

## How are results?

- `li_outcome_summary`: the human verdicts (won, meeting booked, interested, not interested, no outcome). `undecided` means nobody has judged those yet, not that they went badly. Suggest the `answer-replies` skill to close them out.
- `li_outreach_analytics` compares acceptance and reply rates by strategy, ICP or campaign. If it is refused for the user's access, use `li_campaign_status` per campaign.
- An A/B test made with `li_experiment_assign`: `li_outreach_analytics` with `groupBy: "arm"` and `experimentId` (without it, it lists the experiments). Each rate carries `k`, `n` and a 95% interval: say when the intervals overlap, which means no winner yet. `forcedToControl` people (no post) are outside the comparison.
- Invite pace: from `li_list_accounts` (`invitePace`). It is PoliteReach's own per-tier safety policy, not a LinkedIn rule, and cannot be raised. Never quote one flat number for everyone.

## Never contact these people

`li_suppress_profiles` with the `profileUrls` and a `reason`: preview with `confirmSuppress: false`, then `confirmSuppress: true`. Listed people are refused at import, before every invite and before every automated message. Read the list with `li_list_suppressed`; remove one with `li_unsuppress_profile`.

## What is the account doing right now?

`li_account_activity`: what runs now, what is queued, and why anything waits. "Queued" usually means another job is ahead on a shared browser. For setup or sign-in problems, use the `get-started` skill.
