---
name: book-agreed-calls
description: Make sure LinkedIn prospects who agreed to a call actually book it. Records who agreed, matches calendar bookings to prospects, notices cancelled or moved calls, and sends one nudge then one polite close through PoliteReach. Use when the user asks who agreed to a call, who booked, whether someone booked, to follow up on a booking link, or to put booking follow-up on a schedule.
---

# Get agreed calls booked

PoliteReach stores the booking state, matches names, works out who is due and refuses anything over the limits. It reads no calendar and writes no text: you read the user's calendar through the user's own calendar connector and write every word.

## A. Who agreed but has not booked (no calendar needed)

1. Read the replies waiting on the user (`li_replies_to_answer` with `includeThread: true`) and their conversations from the last 30 days (`li_list_conversations`, `li_conversation_history`).
2. List everyone who agreed to a call, with their own words about when.
3. For each one the user approves: `li_track_booking` with `profileUrls`, `agreed: true`, their words as `agreedDayText`, and `agreedDate` (YYYY-MM-DD, their timezone) if they named a day.
4. Anyone not yet sent the booking link: draft a short reply with the plain link for the user's approval. PoliteReach adds the recipient's name to the link by itself.

## B. Match the calendar (needs a calendar connector)

If no calendar connector (such as Google Calendar) is connected, say so and stop. Never guess whether someone booked.

1. **Find bookings.** List events from 7 days ago to 30 days ahead. Keep booking-tool events (cal.com, Calendly, or the user's booking link in the organiser, description, location or title). Ignore meetings with only the user's own team.
2. **Match.** One `li_match_contacts` call with every external attendee: `name`, `email`, `eventId`, `startsAt`, `createdAt`, and the full `description`.
   - `match`: `li_mark_booked` with `calendarEventId` and `meetingStart`. A booked call is `meeting_booked`, never `won`.
   - `ambiguous`: mark nobody; list the candidates for the user.
   - `none`: ignore, unless the attendee gave only one name; then list it for the user.
3. **Cancelled or moved.** `li_bookings` from 7 days ago onward, compared with the calendar.
   - Gone or cancelled: `li_mark_booking_cancelled`.
   - New start time: `li_mark_booked` with the same event id and the new time.

## C. Nudge or close

1. `li_booking_due` (all accounts at once). Read `truncated` and `excluded`; never contact anyone excluded.
2. Per row, from the messages it carries:
   - `nudge`: one line pointing back to what they said about the call, then the plain booking link on its own line. Never propose a time, never pitch.
   - `close`: one line, no link, e.g. "Should I close this off, or still want a slot this week?" in the tone of the thread.
   - Max two short lines, no exclamation marks, no emojis, no "just checking in".
3. If the thread shows they already booked, mark booked instead. If they changed their mind, skip and tell the user.
4. Show the drafts. On a yes (or under a scheduled task's send mode): `li_send_reply` with `confirmSend: true` and `replies`, `bookingNudge: true` for nudges, a separate call with `bookingClose: true` for closes. A refusal names its reason: relay it, never work around it, never retry that person.

## Settings

Booking links and limits are per account: `li_set_booking_settings` (`bookingUrls`, `maxNudges`, `closeAfterDays`, `firstNudgeHours`). Current values are in `li_list_accounts`.

## Put it on a schedule

Follow `references/scheduled-task.md`.
