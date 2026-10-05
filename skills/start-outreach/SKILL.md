---
name: start-outreach
description: Find or research LinkedIn prospects, read their latest posts, and stage their full invite and message sequence in PoliteReach for the user's approval, then launch it. Use when the user pastes LinkedIn profile links, says "reach out to these people" or "find me people who fit", wants to start or add to a LinkedIn campaign, or asks for connection notes and follow-ups written for someone.
---

# Start outreach to these people

PoliteReach sends only text that was written and stored beforehand. Write every word now, show it, and send nothing without the user's yes.

## Before you start

- Load the user's outreach skill (their voice, ICPs, proof, banned words). If they have none, offer the `outreach-skill-builder` skill first; otherwise ask for the offer and voice before writing.
- `li_list_accounts`: pick the sending account. With more than one account, every write must name it (`sessionLabel`). Never guess.
- Note the account tier: only a Premium account sends connection notes. A free account drops them.

## Steps

0. **No list yet?** `li_search_leads` finds people who fit an ICP from LinkedIn's filters (titles, seniority, function, location, industry, headcount, companies, keywords). 1 lookup per person returned: run it with `estimateOnly` first. Results are marked `alreadyContact` / `doNotContact`; drop those. Their links are temporary search ids that the import refuses, so step 2 must look up everyone the user picks.
1. **Look them up first.** `li_contact_dossier` with `profileUrls` (up to 50 per call). Reuse stored research. "Contact not found" means a new person. Flag to the user before going further: anyone with a sequence, anyone mid-conversation, and anyone whose profile is the sending account itself (compare with `profileSlug` from `li_list_accounts`).
2. **Research only what is missing.** `li_enrich_profiles` (costs lookups from the user's allowance; anyone looked up in the last 30 days is free; use `estimateOnly` first for a big list). `companyPage` gives the current employer's industry, size and description; it comes only on a real lookup, never with `estimateOnly`. Use `li_enrich_company` only when a first line needs a specific, recent detail. Never an outside scraper.
3. **Read what they posted.** `li_fetch_posts` (1 lookup per person not read in the last 3 days; reposts excluded). Pick ONE recent post that is their own and worth reacting to; the first touch can then be about something they actually said. `no_posts` means nothing recent, `unreadable` means it could not tell: either way write as usual, about no post.
4. **Match each person to one ICP.** If none fits, say so and suggest who to target instead. Never force a message.
5. **Write the whole sequence.** Connection note (Premium only), first message, then follow-ups and a close: at most four messages in all. Follow the user's skill; default rules:
   - Three short sentences at most; follow-ups two short lines, each adding one new reason.
   - No link in the first message. End on a concrete ask. The close is a yes or no question.
   - No em dashes, exclamation marks, emojis or filler. Never invent a client, number, price or date.
6. **Pick the cadence.** `li_list_strategies`; use one whose steps and delays fit. If none fits, preview `li_create_strategy` (at most 4 message steps) and create it with `confirmCreate: true` only on a yes. Fill only the fields the strategy's steps need: a 3-step strategy (first, followup_1, closer) takes `firstMessage`, `followUp1` and `closer`.
7. **Preview.** `li_ingest_prospects` with `confirmIngest: false`, `launch: false` (so the confirmed call is identical), the account, the strategy and ICP named explicitly (`strategyName` / `icpName`, or `strategyId` / `icpId`), a campaign name, and each prospect's `connectionNote`, `firstMessage`, `followUp1`..`followUp3` / `closer`. For someone with a chosen post, add `engagePostUrl` (and `engageReaction` if not a Like): the account reacts to that post first and the invite waits 20-60 hours. A reaction that cannot run releases the invite at once, so never write copy that only works if the reaction happened. The preview echoes none of the text and returns no per-person checks, only `prospectCount`: print each person's note and messages from your own draft. Who was already approached comes back only on the confirmed call.
8. **Stage on a yes.** Same call with `confirmIngest: true` and `launch: false`. `launch` defaults to true, so always pass false here. Then relay:
   - `collisions`: people already approached from the user's accounts (information, not a refusal).
   - `suppressed`: people on the do-not-contact list, not imported.
   - `result.warnings`: if the campaign is already live, nothing was held back; say so.
9. **Launch on a second yes.** `li_launch_campaign` with that `campaignId` and `confirmLaunch: true`. Warn first if the campaign holds other staged people: a launch starts all of them.

## After launch

- Invites drip inside the account's own pace (from `li_list_accounts`, `invitePace`). That pace is PoliteReach's safety policy, not LinkedIn's, and is not a setting. Never promise a date.
- Messages start once each person accepts, inside the account's sending hours.
- To change or hold queued text later, use the `queued-messages` skill.
