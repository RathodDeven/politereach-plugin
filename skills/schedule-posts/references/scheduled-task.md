# Scheduled task: LinkedIn posts

Create nothing until the user has answered part 1.

## Part 1: ask first

Ask in one message, numbered, defaults in brackets. If their outreach or content skill already answers one, say what you took from it. Ask their timezone if unknown.

1. Which PoliteReach accounts should it cover? Use the names from `li_list_accounts`. [all of them]
2. Which account posts on which weekday? [one post per weekday, your accounts taking turns]
3. What each account posts about. [what your skill says each person is known for]
4. Anchor each post to recent news in that space? [yes, from the last 72 hours, never invented]
5. Anything a post must never say or link, and the one call to action allowed. [never pitch or name your product; at most one soft "DM me" per batch]
6. Schedule them on their own (you can cancel any with `li_cancel_scheduled_post` before it goes out), or show you drafts first? [show you drafts first]
7. What time should posts go out? [weekdays between 18:45 and 19:30, a different minute each time]
8. When should the planning task run, in your timezone? [Sunday and Wednesday at 10:45, each run filling the next few days]

## Part 2: create the task

Create a recurring scheduled task named "LinkedIn posts" at the chosen times, in the user's timezone, with the TASK PROMPT below: CONFIG filled in, the rest word for word. Attach the user's skill or name it in CONFIG.

Tell the user the task name and next run, and remind them to set every tool it uses to always allow. Choosing schedule mode here is the user's standing approval to schedule posts. Offer one run now.

## TASK PROMPT

```text
Write and schedule the next batch of my LinkedIn posts via PoliteReach. Nobody is watching this run: never ask a question, decide and act within MODE.

CONFIG
- Skill: (my outreach or content skill name)
- Grid: (account per weekday)  Post time: (window, my timezone)
- Topics per account: (my answer)  News-anchored: (yes | no)
- Never say: (my list)  Call to action: (my answer)
- MODE: (schedule | draft). In draft mode never call li_schedule_post: put the posts in the report for my yes.

SETUP
1. Load my skill (named in CONFIG) and follow its voice and rules for every word.
2. Make sure the PoliteReach tools (li_*) are loaded. If they are missing, notify me "(task name) failed: PoliteReach not connected" and stop.
3. li_list_accounts. Use the CONFIG accounts; if a name no longer matches, pick the closest. Skip and report any account that is paused or needs signing in again (fix: Reconnect on the PoliteReach Accounts page).

1. li_scheduled_posts: skip any grid slot already filled. Fill the next unfilled slots in the coming 3 days. An unusable account: skip its slot and report it; never move its post to another account's day.
2. One post per slot, text only. If news-anchored, tie it to a real development from the last 72 hours in the space of that account; never invent news, numbers or personal details.
3. Humanize every post: no em dashes, no AI words, no "not X, it is Y", vary the openings, a blank line (\n\n) between paragraphs, bullet lines single-spaced.
4. Schedule each with li_schedule_post on its exact account at a varied minute inside the window, ISO time with my offset (never li_create_post: nothing publishes now). Then li_scheduled_posts to confirm the times and that the blank lines survived; if not, li_cancel_scheduled_post and schedule the fixed text again.

If any tool returns a security-check or sign-in error, stop work on that account and report it; never retry around it.

REPORT: for each post the date, account, time, the news it is tied to and the full text, plus any skipped slot and why, so I can cancel one before it goes out. Notify me once when posts were scheduled or drafted.
```
