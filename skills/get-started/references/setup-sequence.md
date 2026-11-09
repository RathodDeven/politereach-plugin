# Set up PoliteReach, start to finish

For a user who says "set me up" or wants everything ready in one go. Take one step at a time and finish it before starting the next. Say which step you are on. Skip a step only when it is already done, and say so.

## How to ask, in every step

- **Ask with options when you can.** If this app has a tool for asking the user a question with options to pick (for example AskUserQuestion in Claude Code, or a similar ask-with-options tool in the Claude app), use it for every question with a few likely answers: 2-4 options, the recommended or default one first. The user can always type their own answer. Without such a tool, ask numbered questions with the default in brackets, so "defaults" is an answer.
- **Prefill what you know.** Before each round of questions, fill every answer you already know: from memory and earlier chats, the user's connected docs or website, `li_list_accounts` (each account's LinkedIn name `linkedinName` and settings), and a free web search of their company site. Show those answers filled in, each marked with where it came from, and ask the user to confirm or correct them. Ask cold only for what is unknown. Never invent an answer.

## 1. The LinkedIn account

Follow `get-started` (the connector, a LinkedIn account, `li_verify_account`). Stop here until at least one account verifies: nothing else works without one.

## 2. The outreach skill

Every routine writes in the user's voice and judges fit against their ICPs, so this comes before any routine.

- Ask whether they already have an outreach skill (named "(company)-politereach-skill" when PoliteReach built it). If they do, take its exact name, check it has `references/stage-templates.md` and the calendar it uses in `references/company.md` (missing: ask those two parts of the interview now and add them). Then read each account's `booking.urls` in `li_list_accounts`: where one is empty, offer the booking link from the skill's `references/company.md` (or ask for one) and save it with `li_set_booking_settings` on the user's yes. Then move on.
- Otherwise follow `outreach-skill-builder`: the interview, then the build. Note the skill's exact name: every routine attaches it. Its interview also asks which calendar the bookings land on (and checks only that connector), saves the booking link in PoliteReach, and writes the default follow-ups per stage.

## 3. The routines

All optional. Offer the five in one question the user can answer with several picks (a multi-select when the ask-with-options tool allows it; otherwise one numbered line each, "which numbers?"):

1. Replies: drafts answers by default, or sends the clear ones and flags the rest; works best with a calendar connected (default every 2 hours, 10:00 to 20:00).
2. Invitations: shortlists the people who invited the user and fit an ICP by default, or accepts them and starts a short sequence (default 09:00).
3. Bookings: checks who booked after agreeing to a call, then drafts the nudges and the close by default, or sends them; after a call, asks how it went. Needs a calendar connected in this app (default 11:20, 15:20 and 19:20).
4. Posts: writes LinkedIn posts in the user's voice and shows drafts first by default, or schedules them (default planned Sunday and Wednesday).
5. Warm follow-ups: every morning, lists the warm leads whose next touch is due (a follow-up chain step, a next-step date, someone who wrote last, someone gone quiet) with a draft for each, or sends the chain steps they already approved (default daily 09:30).

Set up only the ones they pick, one at a time: each has its own short interview, then creates its scheduled task. Use `answer-replies`, `inbound-invitations`, `book-agreed-calls`, `schedule-posts` and `keep-warm-leads`, each at its `references/scheduled-task.md`. If this app has no scheduled tasks, say so and offer to run a routine's TASK PROMPT once, by hand, in this conversation instead. None picked is fine: every routine can be run by hand later.

## 4. Wrap up

- List what now exists: the skill's name, each account's booking link as `li_list_accounts` shows it now (say "none saved" where it is empty), and each task's name and next run.
- Remind the user to set every PoliteReach tool to always allow: nobody is there to approve a tool while a task runs.
- Offer to start outreach now: they paste LinkedIn profile links and `start-outreach` stages the sequences for their yes.
