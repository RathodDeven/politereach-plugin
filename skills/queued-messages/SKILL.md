---
name: queued-messages
description: Review the LinkedIn messages PoliteReach is about to send and fix or hold queued copy before it goes out. Use when the user asks what goes out today or this week, spots wrong wording in a sequence, wants to change a line across a campaign, hold one message, or pause or resume a campaign.
---

# Review and fix queued messages

Every prospect carries their own copy of the sequence text from the moment they were imported. Changing a strategy later changes nothing for people already in it: fix the queued steps instead.

## What goes out soon

`li_scheduled_steps` with `withinHours: 24` (or a `campaignId`). It lists who, which step, the full text, when, the campaign and the sending account, soonest first. Read `truncated`; raise `limit` rather than assume the page is everything.

Report per account: how many go out, the first few names with times, and any step whose `campaignStatus` is paused. A paused campaign still sends steps that are already scheduled.

## Change nothing without the user's request

- Wrong words for one or a few people: `li_edit_step` with the step ids (from `li_scheduled_steps`, `li_campaign_contacts` or `li_contact_dossier`). Reversible; only the words change.
- The same wrong sentence across a campaign: `li_bulk_edit_steps` with `campaignId`, `find` and `replace`. It previews by default (`dryRun` true). Read `matched` and `contactsUnmatched`: a low match means the find text is not what was imported. Run again with `dryRun: false` once the preview is right.
- A message that should not go at all: `li_cancel_steps` with the step ids. It cannot be undone, so if the message is still wanted, rewrite it with `li_edit_step` instead.
- Confirm with `li_scheduled_steps` again afterwards.

## Pause or resume a campaign

- `li_pause_campaign`: holds the campaign's drip. It does not stop steps already scheduled; cancel or edit those as above.
- `li_resume_campaign`: a campaign that was launched before restarts its drip by itself. Say so before resuming.
- A wrong campaign name or ICP: `li_edit_campaign`. Its strategy and sending account cannot change once it holds people.
