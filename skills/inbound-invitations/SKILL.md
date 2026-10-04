---
name: inbound-invitations
description: Triage the LinkedIn connection requests people sent to the user, accept the ones that fit their ICP and start a short message sequence for them through PoliteReach. Use when the user asks about pending or received LinkedIn invitations, who invited them, which invites to accept, inbound leads, or wants invitation handling on a schedule.
---

# Accept the right invitations

People who invited the user approached first: warm prospects. Accepting is the cheap half; the sequence is the commitment. Accepting does not spend the invite allowance.

## Steps, per account (name the account every time)

1. **Fetch.** `li_received_invitations` with `sessionLabel`, `onlyUnreviewed: true`, `scrollPasses: 40`. It does not cover all accounts at once: with no account it reads only the default one. It returns a `jobId`: read it with `li_job_result` (`jobIds`, `waitMs: 120000`) until done. If `pageState.reachedEnd` is false, read again up to twice, then report that account as incomplete. Connection invites only; never accept a page or newsletter invite.
2. **Check the pipeline.** `li_contact_dossier` with `profileUrls`: anyone already in a campaign is "already in pipeline". Someone who invited two of the user's accounts stays with the account whose lane fits best.
3. **Screen on the headline first.** Students, job seekers, vendors, competitors, clearly outside the ICP: reject. Enrich only the rest: `li_enrich_profiles`, then `li_enrich_company` once per employer if needed.
4. **Decide against the user's outreach skill**, every reason traceable to the data: shortlisted, rejected, or wrong lane (fits, but another account should reach them). Unclear: leave pending. An unanswered invite costs nothing; a wrong accept has no undo.
5. **Show the shortlist** with reasons and the drafted messages. Accept only on a yes (or under a scheduled task's accept mode).
6. **Accept.** `li_accept_invitations` with the shortlisted `profileUrls` and `confirmAccept: true`. Add `wait: true` only for a handful of people; otherwise it returns a `jobId`: read it with `li_job_result` (`jobIds`, `waitMs: 120000`) until done. Read each person's `outcomes` entry. Only people whose outcome is `accepted` move on.
7. **Start their sequence** (if the user wants one). Default: day 2, 7 and 14 after accepting. Day 2 is four short lines: a greeting, one fact from their profile that sets up the offer, the offer in one line, a concrete ask; 300 characters max.
   - Strategy: find "Inbound accepted (Day 2/7/14)" with `li_list_strategies`; create it once with `li_create_strategy` if missing.
   - Campaign: one standing campaign per account and offer, named "Inbound — (account) — (offer)", no date. Find it with `li_list_campaigns` and pass its `campaignId`.
   - `li_ingest_prospects` with `startStage: "message"`, `launch: true`, `confirmIngest: true`, only the people accepted in step 6 (a pending invite is not a connection, and a message-stage send to a non-connection fails for good). Fill location and IANA timezone when known.
   - If `result.campaignCreated` is true, move the rows into the standing campaign with `li_move_contact`. Check the first send is about 48 hours out; if anything is due now, `li_pause_campaign` and tell the user.
8. **Record everyone else.** `li_mark_invitations` with `confirmMark: true`: `rejected` with a one-line reason, `wrong_lane` with `routeTo`, `deferred` for later. This is PoliteReach's record only. It never declines or ignores anything on LinkedIn and adds nobody to do-not-contact.

Never decline, ignore or withdraw an invitation on LinkedIn.

## Report per account

Accepted (name, role, company, why, first message), wrong lane, failed, and "Rejected: N" with short reasons.

## Put it on a schedule

Follow `references/scheduled-task.md`.
