---
name: outreach-skill-builder
description: Interview the user about their company, ideal customers and writing voice, then build their own outreach skill that every PoliteReach workflow writes with. Use when the user is new to PoliteReach, asks for LinkedIn messages written "in my voice", wants to set up or update their ICPs, offers, proof points or banned words, or no outreach skill exists yet.
---

# Build the user's outreach skill

Every PoliteReach workflow writes in the user's voice and judges fit against the user's ICPs. Those facts belong to the user, so they live in a skill the user owns, built from an interview. Invent nothing.

If the `skill-creator` skill is available, use it to build and save the result. Otherwise write the files and tell the user how to add them as a skill in the app they use.

## Part 1: interview (three rounds, one message each)

Wait for answers after each round. Offer options to pick from where you can. Ask again about anything vague.

**Round 1, the business**
1. The company, and what it sells: one line per offer.
2. Two to five ICPs: for each, title, company type, the pain solved, and which offer leads.
3. Who is never contacted (competitors, students, job seekers, others).
4. Proof the user can send: case studies, results, links, and when each fits.
5. The booking link, and the pricing that may be quoted (or "never quote prices").

**Round 2, the voice**
6. Three to five real messages of theirs that got replies.
7. How they write: length, tone, greeting, emoji or not, words they never use.
8. Topics that always go to them instead of being answered (pricing, contracts, legal, anything senior).

**Round 3, how they send**
9. Their PoliteReach accounts (from `li_list_accounts`), whether each is free or Premium, and which offers or ICPs each handles.
10. Cadence. Default: first message the day after they accept, a follow-up on day 5, a close on day 12. For people who invited the user: day 2, 7 and 14.

## Part 2: build it

Name it "(company) outreach". Layout:

- `SKILL.md`: the loop and the hard rules below.
- `references/company.md`: offers, proof, links, pricing that may be quoted.
- `references/icps.md`: each ICP, how to spot them, their pain, their first message, follow-ups and close.
- `references/writing.md`: voice, banned words, message rules.
- `references/objections.md`: common objections and the one line that answers each.
- `references/learned-preferences.md`: empty to start, with a changelog.

Keep ICP definitions inside the skill. PoliteReach stores only an ICP's name and judges nobody against it.

### The loop the built skill follows

1. Look the person up first with `li_contact_dossier` and reuse what is there.
2. Research only what is missing, with `li_enrich_profiles` and `li_enrich_company`. Never an outside scraper.
3. Match them to one ICP. If none fits, say so and name who to target instead. Never force a message.
4. Write the whole sequence at once: a connection note (Premium accounts only; a free account drops notes), a first message, follow-ups and a close. At most four messages; the note is on top of those, so never write a fifth.
5. Stage with `li_ingest_prospects` and `launch: false`, and show every word. Nothing goes out without the user's yes.
6. Apply a one-off correction straight away. For a lasting preference, propose the exact change to `learned-preferences.md` and write it only on a yes.

### Hard rules for every message

- Three short sentences at most; a follow-up is two short lines.
- Sell the problem, not the service. End on a concrete ask, never "let me know".
- No link in the first message. Each follow-up brings one new reason (a proof point, a resource), never a bump.
- The close is a yes or no: "Should I close this off, or (the thing they would still say yes to)?"
- No em dashes, no exclamation marks, no AI words (leverage, streamline, delve, seamless), no "just checking in" or "circling back".
- Never invent a client, a number, a price or a date.
- Someone who agrees to a call gets the plain booking link (PoliteReach adds their name) and is recorded with `li_track_booking`. A booked call is `meeting_booked`, never `won`.
- Topics from answer 8 are drafted for the user, never sent.

Do not build anything that sets the sending pace. Invites drip inside a fixed daily band and weekly ceiling per account tier, and there is no setting for it.

When done, tell the user the skill's exact name: the scheduled routines in this plugin attach it by name.
