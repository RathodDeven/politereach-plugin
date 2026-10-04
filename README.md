# PoliteReach plugin for Claude, ChatGPT and Codex

Run LinkedIn outreach from your own LinkedIn accounts by talking to Claude. Claude researches people, writes every invite note and message in your voice, and decides what to answer. [PoliteReach](https://politereach.com) does the LinkedIn work on its servers: it sends invites inside a per-account safety pace, notices acceptances, sends your pre-written message sequence, reads replies, accepts invitations and publishes posts.

The plugin has two parts:

- **A connector** to the PoliteReach MCP server at `https://mcp.politereach.com/mcp`. You sign in with your PoliteReach account (OAuth). No API key is stored in this plugin.
- **Skills** that teach Claude the PoliteReach workflows: what to read first, what to ask you, and when to stop and wait for your approval.

You need a PoliteReach account with at least one LinkedIn account connected. See [pricing](https://politereach.com/pricing/).

## Install

### Claude Code

```text
/plugin marketplace add RathodDeven/politereach-plugin
/plugin install politereach@politereach
```

Then run `/mcp`, choose `politereach` and sign in.

### claude.ai, Claude Desktop and Cowork

1. Go to **Customize > Plugins > Add > Add marketplace** and enter `https://github.com/RathodDeven/politereach-plugin`.
2. Install **PoliteReach**.
3. Open the plugin's **Connectors** tab, connect PoliteReach, and sign in with your PoliteReach email.

The plugin is not listed in Anthropic's plugin directory yet. Until it is, use the marketplace above.

### Connector only

To use the tools without the skills, add a custom connector with the URL `https://mcp.politereach.com/mcp` and sign in.

### ChatGPT and Codex

The same skills and connector also ship in OpenAI's plugin format ([Agent Plugins](https://agent-plugins.org/)): `plugin.json` and `mcp.json` at the repository root, with the shared `skills/` folder. Nothing is copied: one set of skills serves Claude, ChatGPT and Codex. PoliteReach is not in OpenAI's plugin directory yet.

**ChatGPT today: add the connector by URL.** This gives ChatGPT the PoliteReach tools without the skills.

1. Open **Settings > Security and login** and turn on **Developer mode**. In a Business, Enterprise or Edu workspace, an admin may have to allow it first.
2. Go to [chatgpt.com/plugins](https://chatgpt.com/plugins) and select **+**.
3. Name it `PoliteReach`. Under **Connection**, enter `https://mcp.politereach.com/mcp`.
4. Create it, then sign in with your PoliteReach account when asked (OAuth). Steps from [Connect to ChatGPT](https://developers.openai.com/plugins/deploy/connect-chatgpt).

What works depends on your ChatGPT plan, per [OpenAI's help center](https://help.openai.com/en/articles/12584461-developer-mode-and-mcp-apps-in-chatgpt):

- **Business, Enterprise and Edu:** full tools, including sending (beta). On Business, only workspace admins and owners can use developer mode.
- **Pro:** read-only tools. ChatGPT can report on your pipeline and replies, but cannot send, accept or post.
- Custom connectors work on the web, not in the mobile apps.

**ChatGPT Business or Enterprise: import this repository.** A workspace admin can import it under **Workspace settings > Plugins > Import marketplace**. That brings the skills as well as the connector. ChatGPT reads `.claude-plugin/marketplace.json` from this repository and syncs daily ([OpenAI help](https://help.openai.com/en/articles/20001504-importing-and-syncing-plugin-marketplaces-from-github)). Plugins that use a connector may be marked desktop only.

**Codex:**

```text
codex plugin marketplace add RathodDeven/politereach-plugin
```

Then install **PoliteReach** from the plugin list and sign in. Codex reads `.agents/plugins/marketplace.json` ([Codex plugins](https://developers.openai.com/codex/plugins/build)). This path has not been tested with Codex yet.

**What the skills do there.** The skills are the same as in Claude. They work the same way once the PoliteReach tools are connected. The skill table below applies to every app. Two notes:

- The four routine skills (`answer-replies`, `inbound-invitations`, `book-agreed-calls`, `schedule-posts`) ask your app to create a recurring scheduled task. If your app cannot schedule tasks, run them by hand.
- `book-agreed-calls` needs a calendar connector in the same app (Google Calendar, or Microsoft 365 / Outlook calendar) with its list-events and get-event tools allowed. Without it, it records who agreed but cannot tell who booked, so it sends no nudges.

## Skills

| Skill | What it does |
|---|---|
| `using-politereach` | The overview: what to connect for each job (LinkedIn account, calendar for bookings, scheduled tasks), which skill does what, and every tool with its safety rule. |
| `get-started` | Connects the connector and your LinkedIn account, checks the account really works, and explains its invite pace and settings. |
| `outreach-skill-builder` | Interviews you about your company, ICPs and writing voice, then builds your own outreach skill that every other workflow writes with. |
| `start-outreach` | From LinkedIn profile links: looks each person up, researches what is missing, matches an ICP, writes the whole sequence, stages it for your review and launches on your yes. |
| `answer-replies` | Finds every reply waiting on you, drafts answers, sends the ones you approve, and records how each conversation went. Can run on a schedule. |
| `inbound-invitations` | Screens the invitations people sent you, accepts the ones that fit, and starts a short sequence for them. Can run on a schedule. |
| `book-agreed-calls` | Tracks who agreed to a call, matches bookings on your calendar, and sends one nudge and one polite close. Can run on a schedule. |
| `schedule-posts` | Writes posts in your voice and schedules or publishes them on your accounts. Can run on a schedule. |
| `pipeline-check` | Daily digest, campaign progress, who accepted and what goes out next, pending invites, outcomes, and your do-not-contact list. |
| `queued-messages` | Shows what goes out soon and fixes or holds queued wording before it is sent. |

## How it behaves

- **You approve before anything is sent.** Every tool that sends, accepts, posts or writes previews first. A scheduled routine sends only in the mode you chose when you set it up.
- **Text is written ahead.** PoliteReach sends the words you approved and stored; it never writes messages on its own.
- **Invite pace is fixed per account.** It depends on the account's LinkedIn tier. It is PoliteReach's own safety policy, not a limit LinkedIn publishes, and it cannot be raised.
- **No LinkedIn passwords or cookies in chat.** LinkedIn sign-in happens in a real browser on PoliteReach's server, from a link PoliteReach gives you.
- **Your own LinkedIn accounts only.** Do-not-contact lists are checked before every invite and automated message.

## Data and privacy

This plugin contains only instructions and a connector address. It runs no code on your machine. When you use it, Claude sends the following to PoliteReach at `mcp.politereach.com`:

- LinkedIn profile links of people you ask about, and the notes, messages and posts you approve.
- Your decisions on replies and invitations (outcomes, notes).
- For `book-agreed-calls` only: the name, email, event id, times and description of external attendees on your calendar booking events, read by Claude through your own calendar connector, so PoliteReach can match them to prospects. PoliteReach never connects to your calendar.

Research tools (`li_enrich_profiles`, `li_enrich_company`) use lookups from your PoliteReach allowance.

- Privacy policy: https://politereach.com/privacy/
- Terms of service: https://politereach.com/terms/

## Support

- Contact: https://politereach.com/contact/
- Plugin issues: https://github.com/RathodDeven/politereach-plugin/issues

## License

The files in this repository are MIT licensed. See [LICENSE](LICENSE). The PoliteReach service is covered by its own terms.
