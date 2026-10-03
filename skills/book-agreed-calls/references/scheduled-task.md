# Scheduled task: LinkedIn bookings

First check that a calendar connector (such as Google Calendar) is connected and its tools may run on their own. If not, tell the user to connect it in their connector settings and stop: this task cannot work without it. PoliteReach never connects to the calendar; the assistant reads it and passes the bookings over.

Create nothing until the user has answered part 1.

## Part 1: ask first

Ask in one message, numbered, defaults in brackets. If their outreach skill already answers one, say what you took from it. Ask their timezone if unknown.

1. Which PoliteReach accounts should it cover? Use the names from `li_list_accounts`. [all of them]
2. Your booking link on each account. Check `li_list_accounts`; if one is missing, ask and set it with `li_set_booking_settings`. [what PoliteReach already has]
3. Which calendar do your bookings land on? [your primary calendar]
4. Send the nudge and the close itself, or draft them for you? [draft them for you]
5. The email domain of your own team, so internal meetings are ignored.
6. When should it run, in your timezone? [every day at 11:20, 15:20 and 19:20]

## Part 2: create the task

Create a recurring scheduled task named "LinkedIn bookings" at the chosen times, in the user's timezone, with the TASK PROMPT below: CONFIG filled in, the rest word for word. Attach the user's outreach skill or name it in CONFIG.

Tell the user the task name and next run, and remind them to set every tool it uses to always allow. Choosing send mode here is the user's standing approval to send. Offer one run now.

## TASK PROMPT

```text
Booking follow-through: make sure people who agreed to a call actually book it. Nobody is watching this run: never ask a question, decide and act within MODE.

CONFIG
- Skill: (my outreach skill name)
- Accounts: (my account names)
- Calendar: (which calendar)  Team domain to ignore: (domain)
- MODE: (send | draft). In draft mode never call li_send_reply: put each nudge and close under DRAFTS in the report.

SETUP
1. Load my outreach skill (named in CONFIG) and follow its voice and rules for every word.
2. Make sure the PoliteReach tools (li_*) are loaded. If they are missing, notify me "(task name) failed: PoliteReach not connected" and stop.
3. li_list_accounts. Use the CONFIG accounts; if a name no longer matches, pick the closest. Skip and report any account that is paused or needs signing in again (fix: Reconnect on the PoliteReach Accounts page).
4. Load the calendar tools (list events, get event). If they are missing, notify me "Booking run failed: calendar not connected" and stop. Never guess a booking.

STEP 1: FIND BOOKINGS
On the CONFIG calendar, list events from 7 days ago to 30 days ahead. Keep only booking-tool events (cal.com, Calendly, or my booking link in the organiser, description, location or title). Ignore meetings with only my own team.
For every external attendee pass name, email, eventId, startsAt, createdAt and the full event description to li_match_contacts in one call.
- match: li_mark_booked with calendarEventId and meetingStart.
- ambiguous: do not mark. List under NEEDS CHECK with the candidates.
- none: ignore, unless the attendee gave only one name, then list under NEEDS CHECK.

STEP 2: CANCELLED OR MOVED
li_bookings from 7 days ago onward. Get each event from the calendar.
- Gone or cancelled: li_mark_booking_cancelled.
- Start time changed: li_mark_booked with the same event id and the new time.

STEP 3: NUDGE OR CLOSE
li_booking_due across my accounts. For each row read the messages it carries.
- nudge: one line pointing back to what they said about the call, then my booking link on its own line. Never propose a time, never pitch.
- close: one line, no link: "Should I close this off, or still want a slot this week?" in the tone of the thread.
Max 2 short lines, no exclamation marks, no emojis, no "just checking in". Write the plain booking link; PoliteReach adds their name to it.
Send with li_send_reply, confirmSend true: bookingNudge true for nudges, bookingClose true for closes, one call for each. If a row is refused, drop it and never retry that person.
If the thread shows they already booked, mark booked instead. If they changed their mind, skip and list under NEEDS CHECK.

If any tool returns a security-check or sign-in error, stop work on that account and report it; never retry around it.

REPORT: booked, cancelled or moved, nudged and closed (each message in a code block), NEEDS CHECK. Nothing happened: "No booking actions." and no notification. Otherwise notify me once with the counts and one line per NEEDS CHECK item.
```
