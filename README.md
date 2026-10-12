# Walty skill

An [Agent Skill](https://agentskills.io) for [Walty](https://www.getwalty.com): loyalty cards in Apple Wallet and Google Wallet for small shops, with pushes to every card.

With it, you can run your loyalty card from the assistant you already use. Ask how the card is doing, send a message to every customer who holds it, change the reward, or list the rewards you still have to ship. The skill tells the assistant how to use Walty's tools properly: every push is previewed before it goes out, messages stay short, and the server's limits are respected.

## What you need

1. A Walty account at https://www.getwalty.com. Free to set up.
2. Your API key. In the dashboard open **Assistant** and create one. It starts with `wk_` and is shown once, so copy it straight away. The assistant connection is part of the push plan; on the free plan the server answers 403.
3. An assistant that supports MCP servers and skills: the Claude app, Claude Code, Codex, Grok Build, or another agent. The skill file is the same for all of them.

## Step 1: connect Walty to your assistant

Walty is an MCP server at `https://www.getwalty.com/mcp` (streamable HTTP). Your key goes in a request header.

**Claude app (web, desktop, phone)**

1. Open **Customize → Connectors → + Add → Add custom connector**.
2. Name: `Walty`. URL: `https://www.getwalty.com/mcp`.
3. Authentication: **No sign in**.
4. Request headers: name `x-api-key`, value your key.
5. Click **Add**.

**Claude Code**

```
export WALTY_KEY=wk_YOUR_KEY   # or put it in your shell profile
claude mcp add --transport http walty https://www.getwalty.com/mcp --header "Authorization: Bearer $WALTY_KEY"
```

**Codex**: add to `~/.codex/config.toml`

```toml
[mcp_servers.walty]
url = "https://www.getwalty.com/mcp"
http_headers = { "x-api-key" = "wk_YOUR_KEY" }
```

**Grok Build** (xAI's coding agent)

```
export WALTY_KEY=wk_YOUR_KEY
grok mcp add --transport http walty https://www.getwalty.com/mcp --header "Authorization: Bearer $WALTY_KEY"
```

or in `~/.grok/config.toml`:

```toml
[mcp_servers.walty]
url = "https://www.getwalty.com/mcp"
headers = { "x-api-key" = "${WALTY_KEY}" }
```

`grok inspect` lists the loaded servers; `grok mcp doctor` is the first thing to run if Walty is missing.

**Grok app (grok.com, iOS, Android)**: custom connectors are added at grok.com/connectors → New Connector → Custom, with the server URL. Walty needs a key in a request header, and xAI's docs don't say whether the Custom connector can send one, so this path is untested. If it only offers a sign-in flow, wait for Walty's OAuth sign-in, which is on the way for ChatGPT and will cover Grok too.

**Any other MCP client**: streamable HTTP to the URL above, with the key as `x-api-key` or as `Authorization: Bearer`.

## Step 2: install the skill

The skill is the `SKILL.md` file in this repo. Put it where your assistant loads skills from.

**Claude Code**, for every project:

```
git clone https://github.com/adrianTroop/walty-mcp-skill ~/.claude/skills/walty
```

For one project only, clone it into `.claude/skills/walty/` inside that project instead.

**Grok Build**, for every project:

```
git clone https://github.com/adrianTroop/walty-mcp-skill ~/.grok/skills/walty
```

For one project only, clone it into `.grok/skills/walty/` at the repo root. Skills load when a session starts, so open a new one after installing.

**Other agents**: copy this folder into the directory your skills loader reads, or point the loader at it. The folder name should be `walty`.

The Claude app loads skills from Customize → Skills; upload or paste `SKILL.md` there if your plan offers it. Without the skill the connector still works, the assistant just has fewer rules to go by.

## Step 3: use it

Talk to your assistant in plain words. Some things to try:

- "How's my loyalty card doing this week?"
- "Tell everyone the croissants are out of the oven."
- "Remind the people who have a free coffee waiting. Keep it up for three days."
- "Change the reward text to: a free cake slice, show your card at the till."
- "Which rewards do I still have to send out?"

What happens behind the scenes:

- **Stats** come straight back: cards in wallets, scans this week, how many customers are ready for a reward, pushes left today.
- **Pushes** are shown to you first. The assistant drafts a line of 60 characters or fewer, asks Walty for a preview with the audience count, and sends only after you say yes. Walty allows three pushes a day and the push leaves the card at the time you choose, tonight by default.
- **"Everyone" or "those ready for a reward"**: say which. The assistant picks the audience from your words.
- **The reward text** on the back of the card is separate from pushes. Changing one never changes the other.
- **Several cards?** Name the card in your request, or the assistant will ask.

## What the server enforces, whatever the assistant does

- A missing or wrong key gets a 401. One key per account; making a new one in the dashboard replaces the old one at once, and the dashboard shows when the key was last used.
- Pushes: 60 characters at most, 3 a day, a push subscription, and the message leaves the card after a week at the latest.
- The offer text: 300 characters at most.
- Marking a reward sent only works once per reward and only on the account's own cards.
- Every write is logged on the server.

What the server cannot enforce is the preview step. The MCP protocol has no way for a server to know a human said yes, so "preview, then confirm" is a rule the assistant follows, and the Claude app's own tool-approval prompt is the real gate. Keep tool approval on for `walty_send_push`.

## Troubleshooting

- *"No cards" or an empty answer*: the key is wrong or expired. Make a new one under Assistant in the dashboard and update the header.
- *A push is refused*: you have sent three today, or the shop has no push plan. The assistant will tell you which.
- *The assistant sends without asking*: it isn't using this skill. Check that `SKILL.md` sits in the skills folder under a folder named `walty`.

## Licence

MIT. Use it, change it, share it.
