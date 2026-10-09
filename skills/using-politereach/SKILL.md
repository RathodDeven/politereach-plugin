---
name: using-politereach
description: Overview of PoliteReach, the LinkedIn outreach connector. Says what must be connected for each job (the PoliteReach connector, a LinkedIn account, a calendar for bookings, scheduled tasks for routines), which PoliteReach skill handles which job, and what every li_* tool does with its safety rule. Use when the user asks what PoliteReach can do, how to use it, what they need to connect or set up, which tool or skill to use, why something needs a calendar or a subscription, or makes any PoliteReach request that no other PoliteReach skill covers.
---

# Using PoliteReach

PoliteReach sends LinkedIn outreach from the user's OWN LinkedIn accounts. You (the assistant) research, write and decide. PoliteReach's server does the LinkedIn work: invites, acceptance checks, prefilled messages, reading replies, accepting invitations, posts. Its tools are named `li_*`.

## What must be connected, per job

| Job | Needs | If it is missing |
|---|---|---|
| Anything | The PoliteReach connector, signed in with the user's PoliteReach email | claude.ai, Desktop, Cowork: this plugin's **Connectors** tab. Claude Code: `/mcp`, pick `politereach`. ChatGPT or Codex: add `https://mcp.politereach.com/mcp`. No login yet: https://politereach.com |
| Anything that touches LinkedIn | At least one LinkedIn account added on https://app.politereach.com (**Accounts**), signed in, and checked with `li_verify_account` | New account: add it on the Accounts page. Signed out or security check: `li_login_link`, give the user the link |
| Sending, research, posting | A plan that covers the account (trial or paid) | Relay the refusal as it is. If it carries a link, give that link unchanged; never invent one |
| Research (`li_enrich_profiles`, `li_enrich_company`) | Lookups left in the research allowance (shared across the owner's accounts) | `li_enrich_profiles` with `estimateOnly` first. Out of lookups: say so; `li_contact_dossier` still reads stored research for free |
| Booking check (who booked), and the nudges that depend on it | A calendar connector in the SAME app: Google Calendar, or Microsoft 365 / Outlook calendar (a Calendly or Cal.com connector that lists bookings also works). Its list-events and get-event tools must be allowed | Tell the user to connect their calendar in the app's connector settings. Until then, record who agreed but say you cannot tell who booked, and send no nudges. Never guess |
| Booking nudges | The account's booking link, set with `li_set_booking_settings` (`bookingUrls`) | Ask for the link and set it on a yes. Without it no link is noticed as sent |
| Routines that run on their own | The app's scheduled-tasks feature (claude.ai and Cowork: **Scheduled**; Claude Code: `/schedule`; ChatGPT: tasks), with the tools set to always allow | Run the routine by hand when the user asks |
| Writing in the user's voice | The user's own outreach skill | The `outreach-skill-builder` skill |

PoliteReach itself never connects to a calendar or a mailbox. You read those through the user's own connectors and send PoliteReach only what is needed (attendee name, email, event id, times, description).

## Which skill for which job

- Set up, connect or reconnect, account health: `get-started`.
- Build the user's voice, ICPs and offers: `outreach-skill-builder`.
- Reach out to people from LinkedIn links: `start-outreach`.
- Replies waiting on the user, outcomes, a one-time follow-up for a warm thread: `answer-replies`.
- Invitations people sent the user: `inbound-invitations`.
- Who agreed to a call, who booked, nudges: `book-agreed-calls`.
- Posts now or later, a content calendar: `schedule-posts`.
- How things are going, pending invites, do-not-contact: `pipeline-check`.
- What goes out soon, fixing queued wording: `queued-messages`.

Load the matching skill and follow it. Use the tool map below only for jobs none of them covers.

## Tool map

Every account-scoped tool takes `sessionLabel` (label, LinkedIn name, slug or member id from `li_list_accounts`). "Preview until" a flag means the call is a dry run until that flag is true. Plan and lookups left are in `li_list_accounts` (`plan`, `enrichment`).

**Accounts and health**
- `li_list_accounts`: every account with tier, invite pace, hours, booking settings, plan, research allowance, invite notes this month (`inviteNotes`) and any invite hold (`inviteHold`). Start here.
- `li_verify_account`: real sign-in check now; a successful check clears a security-check hold.
- `li_login_link`: link for the user to sign in to LinkedIn on PoliteReach's server. Never ask for cookies.
- `li_session_status`: saved sessions and their status; `verifyLive` checks one in a browser.
- `li_detect_account_tier`: re-read the account's LinkedIn tier. There is no manual tier setting.
- `li_rename_account`: rename an account; an empty name follows the LinkedIn name.
- `li_pause_account`: hold (`paused: true`) or resume all work. For known outages, not a blocked account.
- `li_clear_invite_hold`: lift the invite breaker's hold (`inviteHold`) after someone has checked the account. Never clear it blind.
- `li_set_office_hours`: sending hours for invites and scheduled messages, or each prospect's local hours (`sendWindowMode`). `end` is exclusive.
- `li_set_auto_withdraw`: daily cleanup of old pending invites. Off by default; a withdrawal blocks re-inviting for about three weeks.
- `li_account_activity`: what runs now, what is queued, and why. Read before calling anything stuck.
- `li_job_result`: what queued work did. A timeout is not a failure; check here, never resend.

**Campaigns and import**
- `li_list_strategies` / `li_create_strategy`: sequence templates, at most 4 messages. Create previews until `confirmCreate`.
- `li_list_icps` / `li_create_icp`: ICP labels for comparing results. Create previews until `confirmCreate`.
- `li_ingest_prospects`: the main import, people plus every prefilled message. Preview until `confirmIngest`; set `launch: false` to stage.
- `li_create_campaign`: manual campaign from targets or CSV. Preview until `confirmCreate`.
- `li_launch_campaign`: start a staged campaign (`confirmLaunch`). Starts everyone staged in it.
- `li_pause_campaign`: hold its invites. Already scheduled messages still send.
- `li_resume_campaign`: a previously launched campaign restarts its drip by itself. Say so first.
- `li_edit_campaign`: rename or change ICP. Strategy and account cannot change once it holds people.
- `li_list_campaigns`: campaigns with ids. `li_campaign_status`: counts. `li_campaign_contacts`: per person, paged (read `truncated`).
- `li_move_contact`: move people between campaigns on the same account, history kept. `allInCampaign` previews and needs `expectCount`.
- `li_reset_contact`: put a contact whose automation died back into the sequence.
- `li_check_targets`: who was already approached. Not needed before an import; the import reports `collisions`.
- `li_send_connection_now`: invites only, no messages, queued under the account's pace. Preview until `confirmSend`.

**Queued messages**
- `li_scheduled_steps`: every message with a send time, soonest first.
- `li_accepted_pending_message`: accepted, first message not out yet. `li_due_messages`: steps already due.
- `li_edit_step`: rewrite unsent steps by step id. Reversible.
- `li_bulk_edit_steps`: replace one exact sentence across a campaign. `dryRun` is true until you set it to false.
- `li_cancel_steps`: hold named steps. Cannot be undone; rewrite instead if the message is still wanted.
- `li_mark_stopped`: end a person's sequence for good. Not a hold.

**Sending**
- `li_send_reply`: answer one or many people (`replies`), each from the account the thread is on. Preview until `confirmSend`. Rows with `bookingCheckRequired` need a calendar search first, then `calendarChecked: true` (booked = send nothing, `li_mark_booked`). A job id means queued; never send twice.
- `li_send_message`: one message that waits for the result. Preview until `confirmSend`.
- `li_send_bulk_messages`: many messages from one account in one run. Preview until `confirmSend`; batches of 15 or fewer.

**Inbox and conversations**
- `li_replies_to_answer`: campaign people who replied; `includeThread: true` attaches the thread.
- `li_list_conversations`: live browser read of the inbox, one account per call; `onlyNeedsReply` and `onlyAddressable` give the owed list. `reachedEnd` false = partial.
- `li_conversation_history`: stored thread, instant. Empty means never read, not never wrote.
- `li_read_conversation`: read one thread live. Slow; only when nothing is stored.
- `li_read_conversations_bulk`: live read of many named people in one session; about six at a time.
- `li_sweep_inbox`: check for replies now. Not visible in `li_job_result`.
- `li_contact_dossier`: everything stored about up to 50 people, including bought research. Open before drafting.
- `li_mark_outcome`: the verdict (`won`, `meeting_booked`, `interested`, `not_interested`, `no_outcome`). A demo or a booked call is never `won`.
- `li_mark_reply_handled`: the user did their part; says nothing about how it went.
- `li_mark_replied`: flag a reply you spotted; stops the sequence.
- `li_snooze_reply`: park a thread until a date (`until`). Nothing is sent when it ends.
- `li_mark_reviewed`: record a thread as looked at (skip, vendor pitch) without a verdict.
- `li_marked_conversations`: people with one verdict; `dueOnly` for snoozes that came due.

**Connections and invitations**
- `li_poll_acceptances`: check acceptances now and schedule each first message.
- `li_check_acceptance`: one person's connected or pending state.
- `li_new_connections`: connections not yet in PoliteReach.
- `li_pending_invites`: sent invites still pending. Read `reach`; `partial` or `failed` is not empty.
- `li_withdraw_invites`: withdraw pending invites (`confirmWithdraw`). Blocks re-inviting for about three weeks. `li_withdraw_status`: last run.
- `li_received_invitations`: invitations sent to the user. Returns a job id; read it with `li_job_result`.
- `li_accept_invitations`: accept named people (`confirmAccept`). No undo. Connection invites only.
- `li_mark_invitations`: record rejected, wrong lane or deferred in PoliteReach only (`confirmMark`). Never declines on LinkedIn.
- `li_invitation_decisions`: read past triage decisions. `li_unmark_invitation`: forget one (`confirm`).
- `li_import_invitation_decisions`: one-time import of an old triage ledger. Preview until `confirmImport`.

**Bookings** (you read the calendar; PoliteReach stores state and enforces limits)
- `li_track_booking`: they agreed to a call (`agreed: true`, `agreedDayText`, `agreedDate`). Search the calendar first: already booked = `li_mark_booked` instead.
- `li_match_contacts`: calendar attendees to prospects. Mark only `match`, never `ambiguous`.
- `li_mark_booked`: a call is on the calendar (`calendarEventId`, `meetingStart`); same event, new time is a move.
- `li_mark_booking_cancelled`: the event is gone or cancelled.
- `li_bookings`: booked calls to compare with the calendar.
- `li_booking_due`: who is due a nudge or a close. Never contact anyone in `excluded`.
- `li_set_booking_settings`: booking links, nudge count, close delay, name prefill.
- Nudges and closes go out only through `li_send_reply` with `bookingNudge` or `bookingClose`.

**Posts**
- `li_schedule_post`: post at an exact time the user chose. Cancellable until it goes out.
- `li_create_post`: publish now. Preview until `confirmPost`.
- `li_scheduled_posts`: queued and recent posts; failed ones carry the reason.
- `li_cancel_scheduled_post`: cancel one not yet published. A published post stays.

**Reporting**
- `li_daily_digest`: one account's day; a blocked account leads.
- `li_outcome_summary`: the human verdicts plus booking rates. `undecided` means not judged yet.
- `li_outreach_analytics`: acceptance and reply rates by strategy, ICP, campaign or account.
- `li_connections`: invite list plus acceptance rate. Give `sessionLabel`, or the stats blend accounts.

**Do-not-contact**
- `li_suppress_profiles`: add people. Preview until `confirmSuppress`. Refused at import, invite and every send.
- `li_list_suppressed`: read the list. `li_unsuppress_profile`: remove one.

**Research** (uses lookups from the allowance; anyone looked up in the last 30 days is free)
- `li_enrich_profiles`: who people are plus their current employer. `estimateOnly` shows the cost first.
- `li_enrich_company`: raw web pages about a company for you to read. One lookup per company per 30 days.
- `li_search_leads`: find people who fit an ICP from LinkedIn's search filters. 1 lookup per person returned; `estimateOnly` first. Results carry temporary search ids: run `li_enrich_profiles` on the people picked before importing.
- `li_fetch_posts`: a person's latest posts (up to 3, reposts excluded). 1 lookup per person not read in the last 3 days. `no_posts` is not `unreadable`.

**Engagement**
- `li_react_to_post`: react (Like by default) to one post from a named account. Preview until `confirmReact`. Shares the account's daily reaction cap with campaigns. A timeout returns a job id: never react again.
- In a campaign, `engagePostUrl` on `li_ingest_prospects` reacts to that post first; the invite then waits 20-60 hours.

Operator only, not for normal use: `li_host_health`, `li_selector_canary`, `li_proxies`, `li_check_proxy`, `li_set_account_proxy`, `li_set_user_default_proxy`, `li_set_team_billing`, `li_set_default_pricing_tier`, `li_set_team_complimentary`, `li_set_account_complimentary`, `li_hold_sending` (pauses every LinkedIn write on every account, at most 24 hours).

## Ground rules, everywhere

- **Preview, then confirm.** Show the dry run, get the user's yes, then call again with the confirm flag. A scheduled task acts only in the mode the user chose when setting it up.
- **Prefilled text only.** Every note, message and post is written now and approved. PoliteReach never writes text at send time; the only addition is the recipient's name in the booking link.
- **Invite pace is fixed.** It is PoliteReach's own safety policy per account tier (`invitePace` in `li_list_accounts`), not a LinkedIn rule. No tool or setting raises it. Quote the account's own numbers, never one flat cap.
- **No cookies or passwords in chat.** If the user pastes one, tell them to delete it and change it. Sign-in happens from `li_login_link`.
- **A security check or sign-in error means stop.** Tell the user to reconnect the account. Never retry around it.
- **Never guess the account.** With more than one, every write names it. Two matches is a refusal, not a pick.
- **Relay refusals.** A refusal says why (plan, do-not-contact, limits, paused account). Relay it with any link it carries; never work around it.
- **Your own accounts only.** At most 4 messages per sequence. A free account drops connection notes.
