# Scheduled task: LinkedIn bookings

## Before anything: the calendar connector

This task cannot work without a calendar connector in the same app as PoliteReach:

- Google Calendar, or Microsoft 365 / Outlook calendar (or a Calendly or Cal.com connector that lists bookings).
- Its list-events and get-event (or search-events) tools must be loaded and set to always allow, because nobody is there to approve them during a run.

Load those tools now. If none are there, tell the user: "This task needs your calendar. Connect Google Calendar or Microsoft 365 / Outlook in this app's connector settings, set its tools to always allow, then ask me again." Then stop. PoliteReach never connects to the calendar; the assistant reads it and passes the bookings over.

The task writes with the user's outreach skill, named "(company)-politereach-skill". Find it among their skills first. If theirs has another name, ask which one it is and use that name. If they have none, offer to build it first (the `outreach-skill-builder` skill, or `li_setup_guide` step `outreach_skill`) and stop.

Also needed: the PoliteReach tools (`li_*`), and the app's scheduled-tasks feature (claude.ai and Cowork: Scheduled; Claude Code: `/schedule`; ChatGPT: tasks). Without scheduling, offer to run the TASK PROMPT below once, by hand, in this conversation.

Create nothing until the user has answered part 1.

## Part 1: ask first

Ask in one message, numbered, defaults in brackets. If their outreach skill already answers one, say what you took from it. Ask their timezone if unknown.

1. Which PoliteReach accounts should it cover? Use the names from `li_list_accounts`. [all of them]
2. Your booking link on each account. Check `li_list_accounts`; if one is missing, ask and set it with `li_set_booking_settings`. [what PoliteReach already has]
3. Which calendar connector and calendar do your bookings land on? [the connected one, your primary calendar]
4. Send the nudge and the close itself, or draft them for you? [draft them for you]
5. The email domain of your own team, so internal meetings are ignored.
6. Anyone who books on your calendar but never through LinkedIn (their email). Mark each with `li_ignore_calendar_event` now, so every run skips them. [none]
7. When should it run, in your timezone? [every day at 11:20, 15:20 and 19:20]

## Part 2: create the task

Create a recurring scheduled task named "LinkedIn bookings" at the chosen times, in the user's timezone, with the TASK PROMPT below: CONFIG filled in, the rest word for word. Attach the user's (company)-politereach-skill to the task and put its exact name in CONFIG, so every run loads it.

Tell the user the task name and next run, and remind them to set every tool it uses to always allow, the calendar tools included: nobody is there to approve a tool while it runs. Choosing send mode here is the user's standing approval to send. Offer one run now.

## TASK PROMPT

```text
Booking follow-through: make sure people who agreed to a call actually book it. Nobody is watching this run: never ask a question, decide and act within MODE.

CONFIG
- Skill: (my company)-politereach-skill
- Accounts: (my account names)
- Calendar: (connector: Google Calendar | Microsoft 365 / Outlook | other) (which calendar)  Team domain to ignore: (domain)
- MODE: (send | draft). In draft mode never call li_send_reply: put each nudge and close under DRAFTS in the report.

SETUP
1. Load my outreach skill (named in CONFIG) and follow its voice and rules for every word. If it cannot be loaded, notify me "(task name) failed: skill not found" and stop.
2. Make sure the PoliteReach tools (li_*) are loaded (if this app defers tools, search for them first). If they are missing, notify me "(task name) failed: PoliteReach not connected" and stop.
3. li_list_accounts. Use the CONFIG accounts; if a name no longer matches, pick the closest. Skip and report any account that is paused or needs signing in again (fix: Reconnect on the PoliteReach Accounts page).
4. Load the CONFIG calendar connector's tools (list events, get event or search events). If they are missing, notify me "Booking run failed: calendar not connected. Reconnect it in the connector settings." and stop before step 1: without the calendar, nobody is matched and nobody is nudged. Never guess a booking.

STEP 1: FIND BOOKINGS
On the CONFIG calendar, list events from 7 days ago to 30 days ahead. Keep only booking-tool events (cal.com, Calendly, or my booking link in the organiser, description, location or title). Ignore meetings with only my own team.
For every external attendee pass name, email, eventId, startsAt, createdAt and the full event description to li_match_contacts in one call.
- match: li_mark_booked with calendarEventId and meetingStart. A reply with unchanged true was already recorded: not news for the report.
If li_match_contacts refuses createdAt or description, pass the rest and add "PoliteReach tools are stale, reconnect the connector" to the report.
- ambiguous: do not mark. If li_bookings already records that event id, it is not news: skip it. Otherwise list under NEEDS CHECK with the candidates.
- none: ignore, unless the attendee gave only one name, then list under NEEDS CHECK.
- ignored: marked not from LinkedIn earlier. Skip; never list it.

STEP 2: CANCELLED OR MOVED
li_bookings from 7 days ago onward. Get each event from the calendar.
- Gone or cancelled: li_mark_booking_cancelled.
- Start time changed: li_mark_booked with the same event id and the new time.

STEP 3: NUDGE OR CLOSE
li_booking_due across my accounts. Drop rows from any account not in CONFIG. Read truncated and excluded; never contact anyone excluded. If one of my one-off scheduled tasks already handles a person (their name is in it), leave them to it. For each row read the thread it carries; if it does not show their agreement and everything after it, read li_conversation_history. bookingLinkSentAt null means the link never went out: send it with one line instead of a nudge about it.
- nudge: one line pointing back to what they said about the call. Never propose a time, never pitch. The link: if a message of ours after their last message already has it, point to it ("can you grab a slot from the link above?") and never paste it again. Only when it is not there, put my booking link on its own second line.
- close: one line, no link: "Should I close this off, or still want a slot this week?" in the tone of the thread.
Max 2 short lines, no exclamation marks, no emojis, no "just checking in". Greet with the row's firstName (already case-fixed); never write a name in all capitals. Write the plain booking link with no full stop after it; never add ?name= or any parameter (PoliteReach adds their name), never a short link.
Right before sending, search the calendar again for each row by full name (then first name + company); anyone who booked since step 1 gets li_mark_booked and no message. Send with li_send_reply, confirmSend true, calendarChecked true, naming no account: each goes from the account its booking is on. bookingNudge true for nudges, bookingClose true for closes, one call for each (a batch cannot mix them). If a row is refused, drop it, resend the rest once, and never retry that person. A returned job id means queued.
If the thread shows they already booked, mark booked instead. If they changed their mind, skip and list under NEEDS CHECK.

If any tool returns a security-check or sign-in error, stop work on that account and report it; never retry around it.

REPORT: booked (only new or changed this run), cancelled or moved, nudged and closed (each message in a code block), NEEDS CHECK. Nothing happened: "No booking actions." and no notification. Otherwise notify me once with the counts and one line per NEEDS CHECK item. If a NEEDS CHECK booking did not come from LinkedIn, I can tell you and you mark it with li_ignore_calendar_event.
```
