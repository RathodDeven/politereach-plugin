---
name: schedule-posts
description: Write LinkedIn feed posts in the user's voice and schedule or publish them on their own accounts through PoliteReach. Use when the user wants a LinkedIn post written, posted now or later, a content calendar for one or more accounts, to see or cancel scheduled posts, or to put post planning on a schedule.
---

# Plan and schedule LinkedIn posts

Posting is not capped by PoliteReach. The user's approval is the gate, so never publish or schedule text the user has not approved (or a scheduled task they set up to do it).

## Steps

1. **See what is already queued.** `li_scheduled_posts`. Skip any slot already filled. A failed post carries its reason.
2. **Know the plan.** Which account posts on which day, what each account posts about, the posting window, anything a post must never say, and the one call to action allowed. Take these from the user's outreach or content skill, or ask. Defaults: one post per weekday with the accounts taking turns; never pitch or name the product; at most one soft "DM me" per batch.
3. **Write.** One text post per slot, up to 3,000 characters. If anchoring to news, use a real development from the last 72 hours in that account's space; never invent news, numbers or personal details. Humanize: no em dashes, no AI words, no "not X, it is Y", varied openings, a blank line between paragraphs.
4. **Show every post** with its account, date and time. Change anything the user asks.
5. **On a yes:**
   - Later: `li_schedule_post` with `sessionLabel`, `text` and `scheduledAt` (ISO time with the user's offset, a varied minute inside the window). The time is kept exactly as given.
   - Now: `li_create_post` with `confirmPost: false` to preview, then `confirmPost: true`. Only when the user wants it out right now.
   - Optional image: one public `imageUrl` (jpeg, png, gif or webp).
6. **Confirm.** `li_scheduled_posts` again: check the times and that the blank lines survived.

To cancel a post that has not gone out: `li_cancel_scheduled_post`. A published post cannot be un-posted this way.

## Report

For each post: date, account, time, the news it is tied to, and the full text, plus any skipped slot and why.

## Put it on a schedule

Follow `references/scheduled-task.md`.
