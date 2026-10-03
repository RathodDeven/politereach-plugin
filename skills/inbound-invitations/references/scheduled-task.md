# Scheduled task: LinkedIn invitations

Create nothing until the user has answered part 1.

## Part 1: ask first

Ask in one message, numbered, defaults in brackets. If their outreach skill already answers one, say what you took from it. Ask their timezone if unknown.

1. Which PoliteReach accounts should it cover? Use the names from `li_list_accounts`. [all of them]
2. Who should it accept, and who never? [the ICPs and the never-contact list in your skill]
3. Accept on its own (at most 20 per account per run), or show you a shortlist first? [show you a shortlist first]
4. With several accounts: which offers or people does each handle? [every account handles everything]
5. Start a message sequence for people it accepts? [yes, 3 messages on day 2, 7 and 14 after accepting]
6. When should it run, in your timezone? [every day at 09:00]

## Part 2: create the task

Create a recurring scheduled task named "LinkedIn invitations" at the chosen time, in the user's timezone, with the TASK PROMPT below: CONFIG filled in, the rest word for word. Attach the user's outreach skill or name it in CONFIG.

Tell the user the task name and next run, and remind them to set every tool it uses to always allow. Choosing accept mode here is the user's standing approval to accept. Offer one run now.

## TASK PROMPT

```text
Accept the right LinkedIn invitations and start their sequence via PoliteReach. Nobody is watching this run: never ask a question, decide and act within MODE.

CONFIG
- Skill: (my outreach skill name)
- Accounts and what each handles: (my answer)
- Accept: (who) Never: (who)
- MODE: (accept | shortlist). In shortlist mode accept nothing and import nothing: report the shortlist with each draft for my yes.
- Sequence: (yes, day 2/7/14 | no). Strategy "Inbound accepted (Day 2/7/14)"; create it once with li_create_strategy if li_list_strategies does not have it. Campaigns: one per account and offer, named "Inbound — (account) — (offer)".

SETUP
1. Load my outreach skill (named in CONFIG) and follow its voice and rules for every word.
2. Make sure the PoliteReach tools (li_*) are loaded. If they are missing, notify me "(task name) failed: PoliteReach not connected" and stop.
3. li_list_accounts. Use the CONFIG accounts; if a name no longer matches, pick the closest. Skip and report any account that is paused or needs signing in again (fix: Reconnect on the PoliteReach Accounts page).

4. FETCH: per account, li_received_invitations with sessionLabel, onlyUnreviewed true, scrollPasses 40, connection invites only. Read its jobId with li_job_result (jobIds, waitMs 120000) until done. If pageState.reachedEnd is false, read again up to twice, then mark that account "incomplete" in the report.
5. li_contact_dossier on everyone: anyone already in a campaign is rejected as "already in pipeline". Someone who invited two accounts stays with the account whose lane fits best.
6. SCREEN on the headline first (students, job seekers, vendors, competitors, clearly outside my ICP: reject). Enrich only the rest: li_enrich_profiles, then li_enrich_company once per employer.
7. DECIDE against my skill, every reason traceable to the data: shortlisted, rejected, or wrong lane (they fit, but another account should reach them).
8. ACCEPT (accept mode): li_accept_invitations with the shortlisted profileUrls, confirmAccept true, wait true. Only outcome "accepted" moves on.
9. SEQUENCE (if on): write day 2 / 7 / 14 in the voice of my skill. Day 2 is four short lines: a greeting, one fact from their profile that sets up the offer, the offer in one line, a concrete ask. 300 characters max. li_list_campaigns, then li_ingest_prospects into the matching campaign with startStage "message", launch true, confirmIngest true, the strategy above, location and IANA timezone filled. If result.campaignCreated is true, move the rows with li_move_contact. Check the first message is due about 48h out; if anything is due now, li_pause_campaign and flag it.
10. RECORD everyone else with li_mark_invitations, confirmMark true: rejected with a one-line reason, wrong_lane with routeTo, deferred for someone unclear (leave them pending). This is only our record; it never declines anything on LinkedIn.

MESSAGE RULES (hard)
Max 3 short sentences, one idea per line, blank line between lines. No em dashes, no exclamation marks, no emojis. No filler: no "just checking in", "circling back", "hope this finds you well". Match my skill's voice. End on a concrete ask, never "let me know". Never invent a price, client, metric or date.

If any tool returns a security-check or sign-in error, stop work on that account and report it; never retry around it.

REPORT per account: accepted (name, role, company, why, first message), wrong lane, failed, and "Rejected: N" with short reasons.
Notify me once only when someone was accepted or shortlisted, something failed, or a campaign was paused. Never decline, ignore or withdraw an invitation, never accept a page or newsletter invite.
```
