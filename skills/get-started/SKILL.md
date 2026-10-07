---
name: get-started
description: Connect PoliteReach to the user's AI assistant and to a LinkedIn account, then check the account is ready to send. Use when the user is setting up PoliteReach, asks how to connect or reconnect LinkedIn, an account says it needs signing in again, or they ask whether an account is healthy, stuck, blocked or what it is doing right now.
---

# Get started with PoliteReach

PoliteReach sends from the user's OWN LinkedIn accounts. You (the assistant) decide and write; PoliteReach's server does the LinkedIn work. Its tools are named `li_*`.

## 1. Check the connector

Call `li_list_accounts`.

- Tools missing: the PoliteReach connector is not connected yet.
  - claude.ai or Cowork: open this plugin's **Connectors** tab and connect PoliteReach, then sign in with your PoliteReach email.
  - Claude Code: run `/mcp`, pick `politereach`, and authenticate in the browser.
  - ChatGPT or Codex: add the PoliteReach plugin or a connector with the URL `https://mcp.politereach.com/mcp`, then sign in with your PoliteReach email.
  - No PoliteReach login yet: start at https://politereach.com.
- Tools present: keep the account labels it returns. Every other tool selects an account by that label (`sessionLabel`).

## 2. Connect or reconnect a LinkedIn account

- New account: the user adds it on the PoliteReach dashboard (https://app.politereach.com, **Accounts**), then signs in to LinkedIn in the real browser it opens.
- Existing account that needs signing in again: call `li_login_link` for it and give the user the link. It works at any account status.
- Never ask for, accept or repeat LinkedIn cookies, passwords or one-time codes in chat. If the user pastes one, tell them to remove it and change it; the server sign-in needs none of it.

## 3. Confirm it really works

- `li_verify_account` for the account. Health is otherwise only updated after something fails, so an idle account can look fine while signed out. A pass also clears a security-check hold.
- Read back from `li_list_accounts`, per account, in plain words:
  - Tier and invite pace (`invitePace`). These limits are PoliteReach's own safety policy, set per tier. They are not LinkedIn's published rules and cannot be raised. Quote the account's own numbers; never state one flat cap for everybody.
  - Sending hours, booking link, plan and research allowance.
  - Invite cleanup (auto-withdraw): off unless the user turned it on.

## 4. Optional settings (only when the user asks)

- Sending hours: `li_set_office_hours`. They hold invites and scheduled messages only, never a message the user sends now. `end` is exclusive (20 means up to 19:59).
- Booking link: `li_set_booking_settings` with `bookingUrls`. PoliteReach adds each recipient's name to the link itself.
- Invite cleanup: `li_set_auto_withdraw`. Explain first: a withdrawn invite blocks re-inviting that person for about three weeks and cannot be undone. Turn it on only on a clear yes.

## When something looks stuck

- `li_account_activity`: what is running now, what is queued, and why. Accounts take turns on shared browsers, so "queued" is usually normal.
- A security check or sign-in error: stop. Tell the user to reconnect (step 2). Never retry around it.
- A known outage: `li_pause_account` with `paused: true` holds all work without alerts; `paused: false` resumes. It does not fix a blocked account.
- `inviteHold` on an account: 3 invites in a row failed the same way, so new invites stopped (messages did not). Tell the user what it says. Clear it with `li_clear_invite_hold` only after they have checked the account on LinkedIn.

## Next

1. Build the user's outreach skill: the `outreach-skill-builder` skill.
2. Start outreach to people: the `start-outreach` skill.
3. What else PoliteReach does, and what to connect for it (a calendar for booking checks, scheduled tasks for routines): the `using-politereach` skill.
