---
name: outreach-skill-builder
description: Interview the user about their company, ideal customers and writing voice, then build their own outreach skill, named "(company)-politereach-skill", that every PoliteReach workflow writes with, and set PoliteReach up to match (strategies, ICPs, do-not-contact list, office hours). Use when the user is new to PoliteReach, asks for LinkedIn messages written "in my voice", wants to set up or update their ICPs, offers, proof points or banned words, or no outreach skill exists yet.
---

# Build the user's outreach skill

Every PoliteReach workflow writes in the user's voice and judges fit against the user's ICPs. Those facts belong to the user, so they live in a skill the user owns, built from an interview. Invent nothing. Build only once the user has answered.

If the `skill-creator` skill is available, use it to build and save the result. Otherwise write the files and tell the user how to add them as a skill in the app they use.

## Part 1: interview (three rounds, one message each)

Read `li_list_accounts` first so you can name the user's accounts back to them. Wait for answers after each round. Offer options to pick from where you can. Ask again about anything vague.

**Round 1, the business**
1. The company, and what it sells: one line per offer.
2. Two to five ICPs: for each, title, company type, the pain solved, and which offer leads. And when someone fits two, which one wins.
3. Who is never contacted (competitors, students, job seekers, others). Help define a competitor by a test (for example: do they sell what the user sells, or would they buy it?), and ask whether peers who do the same work are partners or a pass. For specific people, ask for each one's LinkedIn profile link: only a person with a link can go on the do-not-contact list.
4. Proof the user can send: case studies, results, links, and when each fits.
5. The booking link, and the pricing a plain price question may be answered with (or "never quote prices").
6. Where their prospects are (country or region), so invites and messages land in the prospects' working morning.

**Round 2, the voice**
7. Three to five real messages of theirs that got replies (they paste them).
8. How they write: length, tone, greeting, emoji or not, words they never use, and how casual to be with senior people.
9. Topics that always go to them instead of being answered (pricing beyond answer 5, contracts, legal, anything senior).

**Round 3, how they send**
10. Their PoliteReach accounts (from `li_list_accounts`), whether each is free or Premium, which offers or ICPs each handles, and which account writes to a second person at the same company. Default: every account handles everything.
11. Cadence. Default: three messages, the first the day after they accept, a follow-up on day 5, a close on day 12. For people who invited the user: day 2, 7 and 14. Four messages is the most PoliteReach sends.

## Part 2: build it

Name it "(company)-politereach-skill": the company name in lowercase, words joined by hyphens (for example acme-politereach-skill). Every PoliteReach routine attaches it by that exact name. Layout:

- `SKILL.md`: the loop and the hard rules below, and which reference file to read at which step. Keep it short: one home for each fact.
- `references/company.md`: offers, proof, links, pricing that may be quoted.
- `references/icps.md`: an ordered table of signals (first match wins) with the tie-break, then per ICP: how to spot them, seniority, their pain, the offer, the proof, who is NOT this ICP, the account that writes to them, and their first message, follow-ups and close.
- `references/writing.md`: voice, banned words, message rules.
- `references/objections.md`: per objection, what they say, what it really means, and the one line or question that moves it. Plus a step-down ladder (after a no, one other offer per reply, never one they already heard) and a graceful close.
- `references/learned-preferences.md`: empty to start, with a changelog.

Keep ICP definitions inside the skill. PoliteReach stores only an ICP's name and judges nobody against it.

### The loop the built skill follows

1. Read `li_list_accounts` every run, never from memory: which accounts are usable or held, and `inviteNotes`.
2. Look the person up first with `li_contact_dossier` and reuse what is there. It needs the profile link: ask for one if given only a name. If they already replied, read the thread instead of writing a sequence.
3. Research only what is missing, with PoliteReach's own lookups: `li_research_profiles` for a list (`estimateOnly` first, it shows the cost), `li_enrich_profiles` and `li_enrich_company` for one person, `li_fetch_posts` for their recent posts. With no list, `li_search_leads` finds people; drop anyone already a contact or on the do-not-contact list. A web search for company news is fine; never an outside scraper. Lookups cost credits, so screen on the headline before paying.
4. Match them to one ICP. If none fits, say so and name who to target instead. Never force a message.
5. Write the whole sequence at once: a connection note, a first message, then the follow-ups and close the cadence has. Write the note only when the account sends notes (`inviteNotes`: a free account drops notes, and Premium notes can run out for the month); otherwise write none and say so. At most four messages, three by default; the note is on top of those, so never write a fifth. You may pick ONE recent post of theirs to like before the invite (`engagePostUrl`), but never write a message that only works if they saw the like.
6. Preview with `li_ingest_prospects` (`confirmIngest: false`) and show every word, the account and the campaign. Run `li_list_campaigns` first. Import into the standing campaign by `campaignId` (one per account and offer, never one per batch or per day) only when the user's yes covers sending now: it is live, so the import sends. Otherwise use a new campaign that is not launched. Every row names `strategyName`, `icpName` and the account, and carries location, an IANA timezone and what you learned (`whyTheyFit`, `whatTheySell`, `context`), so the next lookup reuses it. Someone the user is already connected to starts at `startStage: "message"`.
7. On the user's yes, stage with `confirmIngest: true` and `launch: false`. Nothing goes out without the user's yes. `launch: false` holds nothing in a campaign that is already live: there the import sends, so say so before, and check each row's first send time after. For more than ten people, tell the user the pace and roughly when the last invite goes out.
8. A warm thread that went quiet: offer a one-off scheduled task for the next touch, which sends nothing if they replied in the meantime.
9. Apply a one-off correction straight away. When the user makes the same choice two or three times, or says "always", propose the exact change to `learned-preferences.md` and write it only on a yes. Edit the old rule in place instead of adding one that contradicts it. Whenever the skill changes, give the user the whole skill folder back, never only the changed file.
10. Every few weeks, compare ICPs and strategies with `li_outreach_analytics` and `li_outcome_summary`. To test two versions of a message, `li_experiment_assign` splits people fairly.

### Hard rules for every message

- A first message is three short sentences at most: 150 to 200 characters, 300 at most counting the greeting and line breaks. A follow-up is two short lines. One idea per line, a blank line between, written into the text itself.
- Line 2 of a first message is one fact you can point to on their profile that sets up the offer. Never a guess about who they sell to, never filler (founding year, years in the role, follower count).
- A post of theirs is mentioned by its topic in a few words and a short true reaction, never its numbers or claims.
- Sell the problem, not the service. The first message offers one named thing they can say yes or no to; the call ask starts at the first follow-up or their first positive reply. Never end on "let me know".
- No link in the first message. Each follow-up brings one new reason (a proof point, a resource), never a bump.
- The close is a yes or no: "Should I close this off, or (the thing they would still say yes to)?"
- Cold messages never mention limits, safety, reply rates or prices.
- No em dashes, no exclamation marks, no AI words (leverage, streamline, delve, seamless), no "great question", no "just checking in" or "circling back".
- Never invent a client, a number, a price or a date.
- Check the exact text against these rules before showing or sending it.
- Someone who agrees to a call: check the user's calendar first, people book within minutes. Not booked: send the plain booking link (PoliteReach adds their name) and record it with `li_track_booking`. A booked call is `meeting_booked`, never `won`.
- Topics from answer 9 are drafted for the user, never sent: leave the thread in their queue with `li_mark_replied` and a note starting "ACTION NEEDED".

Do not build anything that sets the sending pace. Invites drip inside a fixed daily band and weekly ceiling per account tier, and there is no setting for it.

## Part 3: set up PoliteReach to match

Show the user each of these and do it on their yes:

- `li_list_strategies`, then `li_create_strategy` for each cadence that has none (first step delay 0), and `li_create_icp` for each ICP. An import that names no strategy falls back to a default sequence, not the user's cadence.
- `li_suppress_profiles` for the people from answer 3 who came with a profile link (preview first, then `confirmSuppress`), so PoliteReach refuses them at every import and send. Competitors and other categories stay rules in the skill.
- `li_set_office_hours` on each account for the region in answer 6. Keep the account's own `timezone` (it is also the browser's timezone). Prospects in several zones: `sendWindowMode: "prospect"` (invites 8-11, messages 8-18 in their time). One market: convert its morning into the account's local hours and change only `start` and `end`.

When done, tell the user the skill's exact name, "(company)-politereach-skill" with their company filled in: the scheduled routines in this plugin attach it by that name. If they keep their outreach skill under another name, tell them that name instead, so the routines use it.
