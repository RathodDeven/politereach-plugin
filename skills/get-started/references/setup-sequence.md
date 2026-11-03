# Set up PoliteReach, start to finish

For a user who says "set me up" or wants everything ready in one go. Take one step at a time and finish it before starting the next. Say which step you are on. Skip a step only when it is already done, and say so.

## 1. The LinkedIn account

Follow `get-started` (the connector, a LinkedIn account, `li_verify_account`). Stop here until at least one account verifies: nothing else works without one.

## 2. The outreach skill

Every routine writes in the user's voice and judges fit against their ICPs, so this comes before any routine.

- Ask whether they already have an outreach skill (named "(company)-politereach-skill" when PoliteReach built it). If they do, take its exact name and move on.
- Otherwise follow `outreach-skill-builder`: the interview, then the build. Note the skill's exact name: every routine attaches it.

## 3. The routines

Offer the four routines in one message, one line each, and ask which to set up:

1. Replies: drafts answers by default, or sends the clear ones and flags the rest; works best with a calendar connected (default every 2 hours, 10:00 to 20:00).
2. Invitations: shortlists the people who invited the user and fit an ICP by default, or accepts them and starts a short sequence (default 09:00).
3. Bookings: checks who booked after agreeing to a call, then drafts one nudge and one close by default, or sends them. Needs a calendar connected in this app (default 11:20, 15:20 and 19:20).
4. Posts: writes LinkedIn posts in the user's voice and shows drafts first by default, or schedules them (default planned Sunday and Wednesday).

Set up only the ones they pick, one at a time: each has its own short interview, then creates its scheduled task. Use `answer-replies`, `inbound-invitations`, `book-agreed-calls` and `schedule-posts`, each at its `references/scheduled-task.md`. If this app has no scheduled tasks, say so and offer to run a routine's TASK PROMPT once, by hand, in this conversation instead.

## 4. Wrap up

- List what now exists: the skill's name, and each task's name and next run.
- Remind the user to set every PoliteReach tool to always allow: nobody is there to approve a tool while a task runs.
- Offer to start outreach now: they paste LinkedIn profile links and `start-outreach` stages the sequences for their yes.
