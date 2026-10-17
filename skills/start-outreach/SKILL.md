---
name: start-outreach
description: Find or research LinkedIn prospects, read their latest posts, and stage their full invite and message sequence in PoliteReach for the user's approval, then launch it, including lists split across several campaigns, accounts or A/B test arms. Use when the user pastes LinkedIn profile links or lead lists, says "reach out to these people" or "find me people who fit", wants to start or add to LinkedIn campaigns, wants to test strategies against each other, or asks for connection notes and follow-ups written for someone.
---

# Start outreach to these people

PoliteReach sends only text that was written and stored beforehand. Write every word now, show it, and send nothing without the user's yes.

## Before you start

- Load the user's outreach skill (their voice, ICPs, proof, banned words). If they have none, offer the `outreach-skill-builder` skill first; otherwise ask for the offer and voice before writing.
- `li_list_accounts`: pick the sending account. With more than one account, every write must name it (`sessionLabel`). Never guess.
- Note the account tier: only a Premium account sends connection notes. A free account drops them.
- Check `inviteNotes` in `li_list_accounts`. Premium can also have a monthly note allowance. When `state` is `spent` (or `not_sent_on_free`), write no connection notes; `guidance` says until when: LinkedIn will not deliver them. The invites still go out, and messages are unaffected.

## Steps

0. **No list yet?** `li_search_leads` finds people who fit an ICP from LinkedIn's filters (titles, seniority, function, location, industry, headcount, companies, keywords). 1 lookup per person returned: run it with `estimateOnly` first. Results are marked `alreadyContact` / `doNotContact`; drop those. Their links are search ids (`/in/ACwAA…`). Pass them to research, checks and import as they are: every tool resolves each to the public profile URL first and says so in `searchIds`. An id it could not resolve is not imported; relay its `reason`.
1. **Research the whole list in one call.** `li_research_profiles` with every `profileUrl` (up to 500), `include` `pipeline`, `profile`, `company`, plus `posts` when the first touch may be about a post. Run `estimateOnly: true` first on a big list and show the cost. It returns one row per person: every campaign they are already in, who they are, their employer, their latest posts. Someone already in the pipeline is reused free. A list that takes longer than about 20 seconds returns a `jobId`: read it with `li_job_result` (`jobId`, `waitMs`, then `offset` = `nextOffset` until it is null). Several separate lists: research them in parallel calls, not one after another. Do not rebuild this from `li_check_targets`, `li_enrich_profiles` and `li_fetch_posts` joined by URL.
   - Flag to the user before going further: anyone with `pipeline.memberships` (already in a campaign; `hasInbound` = mid-conversation), and anyone whose profile is the sending account itself (compare with `profileSlug` from `li_list_accounts`).
   - `pipeline.unreadable`, or a `status` of `failed` or `pending`, means not known, never "none".
   - For one person's stored notes and conversation, `li_contact_dossier`.
2. **Company detail only when needed.** `li_enrich_company` when a first line needs a specific, recent detail. Never an outside scraper.
3. **Pick a post.** From the research `posts`: ONE recent post that is their own and worth reacting to; the first touch can then be about something they actually said. Only a post with `reactable: true` can be an `engagePostUrl` (a group post is not). `no_posts` means nothing recent, `unreadable` means it could not tell: either way write as usual, about no post.
4. **Match each person to one ICP.** If none fits, say so and suggest who to target instead. Never force a message.
5. **Write the whole sequence.** Connection note (Premium only, and not while `inviteNotes.state` is `spent`), first message, then follow-ups and a close: at most four messages in all. Follow the user's skill; default rules:
   - Three short sentences at most; follow-ups two short lines, each adding one new reason.
   - No link in the first message. End on a concrete ask. The close is a yes or no question.
   - No em dashes, exclamation marks, emojis or filler. Never invent a client, number, price or date.
6. **Pick the campaigns.** `li_list_campaigns` first, compact: `namePrefix` and `sessionLabel` to narrow it, `fields: ["name", "accountLabel", "contactCount", "launchedAt"]`, and `nextOffset` as `offset` while `truncated`. Note each campaign's account, exact name and `launchedAt`. A campaign that is already launched sends whatever is imported into it; `launch: false` cannot stage people there. For a batch the user has not approved, use a new campaign name. To add to an existing campaign, pass its `campaignId`: a name that is not an exact match creates a second campaign.
7. **Pick the cadence.** `li_list_strategies`; use one whose steps and delays fit. If none fits, preview `li_create_strategy` (at most 4 message steps) and create it with `confirmCreate: true` only on a yes. Fill only the fields the strategy's steps need: a 3-step strategy (first, followup_1, closer) takes `firstMessage`, `followUp1` and `closer`.
8. **Preview.** `li_ingest_prospects` with `confirmIngest: false`, `launch: false` (so the confirmed call is identical), the account, the strategy and ICP named explicitly (`strategyName` / `icpName`, or `strategyId` / `icpId`), the campaign (`campaignId`, or a new `campaignName`), and each prospect's `connectionNote`, `firstMessage`, `followUp1`..`followUp3` / `closer`.
   - **Many campaigns, ONE call.** A list split by ICP, account or lane goes in one call of up to 500 rows: put `sessionLabel`, `campaignName` (or `campaignId`), `strategyName` and `icpName` on each row; a row leaving one out takes the call's. Never one call per campaign. The preview's `campaigns` lists each campaign it will fill, with its account and count.
   - **Shared copy.** When the follow-ups are the same for everyone, send them once as `defaultSteps` (`firstMessage`, `followUp1`..`followUp3`, `closer`, `connectionNote`) and keep only the personal lines on each row. `{first}` is the only variable (the row's `firstName`, else the first word of `name`). Read `copy`: it lists every row over a limit (a note over 300 is shortened) and any row refused for having no name.
   - Fill `location` and `timezone` (IANA zone from the location: city if known, else the country's main zone; blank if unknown) on every row. Accounts can send in each prospect's local hours, and that needs the zone.
   - Store the research too, so later drafts reuse it: `name`, `headline`, `role`, `company`, `companyDomain`, `icpBucket`, `whyTheyFit`, `whatTheySell`, `context`.
   - `startStage`: `connection` (default) invites first. Someone already connected to that account gets `message`: their first message is scheduled with no invite. Never `connection` for a connection. For someone with a chosen post, add `engagePostUrl` (and `engageReaction` if not a Like): the account reacts to that post first and the invite follows about a day later, in the person's own morning window (or the account's sending hours; 18-30 hours with no window to aim at). A reaction that cannot run releases the invite at once, so never write copy that only works if the reaction happened. The preview echoes none of the text and returns no per-person checks, only `prospectCount`: print each person's note and messages from your own draft. Who was already approached comes back only on the confirmed call.
9. **Stage on a yes.** Same call with `confirmIngest: true`, `launch: false` and an `idempotencyKey` (any unique string). `launch` defaults to true, so always pass false here. After a timeout, 502 or 503, call again with the SAME key: a call that finished returns its stored answer instead of importing twice. Then relay:
   - `campaigns`: each campaign touched, created or existing, with imported, updated and rejected counts. `routed` rows not imported carry the reason.
   - `collisions`: people already approached from the user's accounts (information, not a refusal).
   - `suppressed`: people on the do-not-contact list, not imported.
   - `result.warnings`: if the campaign is already live, nothing was held back; say so.
   - Read back `result.campaignName`, `result.campaignCreated` and `result.campaignLaunchedAt` before saying "staged, nothing sent". Rows that landed in the wrong campaign: `li_move_contact` (same account only), and tell the user.
10. **Launch on a second yes.** `li_launch_campaign` with that `campaignId` and `confirmLaunch: true`. Several campaigns: ONE call with `campaignIds` (from step 9's `campaigns`); without `confirmLaunch` it lists them with their people and anything it would refuse, then the same call with `confirmLaunch: true` launches each. Warn first if a campaign holds other staged people: a launch starts all of them.

## Testing strategies against each other

Never deal people into lanes by hand. `li_experiment_assign` does it on the server:
- First call: `experiment` with a `name`, 2-8 `arms` (`key`, `strategyName`, `engage`), optionally `requirePostForArms` and a `controlArm` for people without a reactable post. Later calls: `experimentId`. People in an arm stay there.
- Each row carries its account and ICP (or the call's), its own text, and `engagePostUrl` when it has a reactable post. Shared text per arm goes in `armSteps`, shared text for every arm in `defaultSteps`.
- Preview with `confirmAssign: false` (shows the split per ICP and account and the campaigns), then `confirmAssign: true` with an `idempotencyKey`. Everything is staged.
- Launch with `li_launch_campaign` and the reply's `launch.campaignIds`. Results: `li_outreach_analytics` with `groupBy: "arm"` and `experimentId`. Quote each rate with its `n` and interval; small arms decide nothing.

## Fixing a staged batch

- A new lane with nobody in it yet: `li_create_campaign` with no targets creates an empty campaign.
- A staged campaign on the wrong strategy: `li_edit_campaign` with `strategyId` and `resnapshot: true` (`dryRun` first). Works only while nobody in it has been invited or sent anything; it names anyone missing text for a new step.
- People in the wrong lane: `li_move_contact` (same account only), with `resnapshot: true` to take the target campaign's steps.

## After launch

- Invites drip inside the account's own pace (from `li_list_accounts`, `invitePace`). That pace is PoliteReach's safety policy, not LinkedIn's, and is not a setting. Never promise a date.
- Messages start once each person accepts, inside the account's sending hours.
- To change or hold queued text later, use the `queued-messages` skill.
