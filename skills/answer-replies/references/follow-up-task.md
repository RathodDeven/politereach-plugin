# One-time follow-up for a warm thread

For one person whose thread has nothing scheduled: they replied, a call happened, a proposal went out, or you are waiting on them. The run happens days later with no memory of this chat, so the prompt must carry everything.

## Part 1: settle it with the user

1. When: about 7 days after our last message, at a morning hour in the user's timezone, unless they named a date. If a follow-up task for this person already exists, update it instead of making a second one.
2. Send or draft. [draft]
   - Draft: the run writes the message and sends nothing.
   - Send: only the exact text the user approves now, stored word for word in the prompt. Text written during the run is never sent.

## Part 2: create the task

Create a one-time scheduled task named "LinkedIn follow-up: (name)" at that time, with the TASK PROMPT below, every field filled. Tell the user when it runs. Remind them to set the PoliteReach tools to always allow when it is in send mode.

## TASK PROMPT

```text
One LinkedIn follow-up via PoliteReach. Nobody is watching this run: never ask a question, decide and act within MODE.

PERSON
- Name: (name)
- Profile URL: (profile URL)
- Sending account: (sessionLabel)
- What we offer them: (one line)
- Skill: (my outreach skill name), load it first and follow its voice.

THE THREAD SO FAR
(the thread in a few lines, oldest first, with dates: what was said, what was agreed, what was sent, who waits on whom)
Our last message: (ISO timestamp). Anything from them after it is new.
Corrections I made while drafting: (list, or none)

MODE: (send | draft)
Approved text (send mode only, sent word for word): (text)

STEPS
1. Make sure the PoliteReach tools (li_*) are loaded. If they are missing, notify me "(task name) failed: PoliteReach not connected" and stop.
2. Read the thread: li_read_conversation with the profile URL and sessionLabel; if that fails, li_conversation_history.
3. Decide on the newest message:
   a. They wrote after our last message: send nothing. Report their message word for word and a suggested reply.
   b. They said no or asked us to stop: send nothing. Report it.
   c. Nothing new: in send mode, li_send_message with the approved text, confirmSend true, once. In draft mode, write the follow-up in my voice and report it.
4. If they agreed to a call anywhere in the thread, search my calendar for them first (if connected). Booked: send nothing, li_mark_booked, report it.

MESSAGE RULES (hard): max 3 short sentences, no em dashes, no exclamation marks, no emojis, no "just checking in" or "circling back", never re-pitch what the thread already covered, never invent a fact.

If a tool returns a security-check or sign-in error, stop and report it; never retry around it.

REPORT: which case (a, b or c), their newest message with its time, and the message sent or drafted in a code block. Notify me once.
```
