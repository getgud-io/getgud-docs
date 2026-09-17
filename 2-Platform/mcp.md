# Getgud MCP Server

The Getgud [Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction) server connects your AI app to your Getgud account. Once connected, you can ask Claude, ChatGPT, Cursor or any other MCP client about your titles, matches, players, insights and reports in plain language, and let it manage your Getgud setup for you.

It is a hosted server, so there is nothing to install.

```
https://mcp.getgud.io/mcp
```

The AI app acts as **you**. It can see and do exactly what your dashboard user can, within the permissions of your role, and nothing more.

## Contents

- [Connect your AI app](#connect-your-ai-app)
- [Log in and permissions](#log-in-and-permissions)
- [What you can ask](#what-you-can-ask)
- [Tools](#tools)
- [Working with match data](#working-with-match-data)
- [Security and privacy](#security-and-privacy)
- [FAQ](#faq)

## Connect your AI app

Every client follows the same two steps: add the server URL, then log in to Getgud in the browser tab that opens.

### Claude (web and desktop)

1. Open **Settings > Connectors**.
2. Click **Add custom connector**.
3. Name it `Getgud` and paste `https://mcp.getgud.io/mcp` as the URL.
4. Click **Connect** and log in to Getgud when prompted.

### Claude Code

```bash
claude mcp add --transport http getgud https://mcp.getgud.io/mcp
```

Then run `/mcp` inside a Claude Code session and pick `getgud` to log in.

### ChatGPT

1. Open **Settings > Apps & Connectors** and turn on **Developer mode** under the advanced settings.
2. Create a new connector, name it `Getgud` and paste `https://mcp.getgud.io/mcp` as the MCP server URL.
3. Choose OAuth as the authentication method and log in to Getgud when prompted.

### Cursor

1. Open **Cursor Settings > Tools & Integrations** and click **New MCP Server**.
2. Add Getgud to `mcp.json`:

```json
{
  "mcpServers": {
    "getgud": {
      "url": "https://mcp.getgud.io/mcp"
    }
  }
}
```

3. Save. Cursor shows **getgud** under MCP Tools and asks you to log in on first use.

### Visual Studio Code

1. In the command palette run `MCP: Open User Configuration`.
2. Add Getgud to `mcp.json`:

```json
{
  "servers": {
    "getgud": {
      "type": "http",
      "url": "https://mcp.getgud.io/mcp"
    }
  }
}
```

### Codex

```bash
codex mcp add getgud --url https://mcp.getgud.io/mcp
```

Codex opens the Getgud login for you. If it does not, run `codex mcp login getgud`.

### Other clients

Any client that supports remote MCP servers over Streamable HTTP with OAuth works: add `https://mcp.getgud.io/mcp` as the server URL. For a client that only supports local servers, bridge it with [`mcp-remote`](https://github.com/geelen/mcp-remote):

```json
{
  "mcpServers": {
    "getgud": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://mcp.getgud.io/mcp"]
    }
  }
}
```

## Log in and permissions

When you connect, a browser tab opens on the Getgud dashboard and asks **"Allow <your AI app> to access your Getgud account?"**. Enter your organization name, email and password, then click **Allow**. Click **Deny** to cancel.

- The AI app gets the same access as your dashboard user. If your role cannot delete players in the dashboard, the AI app cannot either.
- The connection can **change and delete** data, not only read it. Tools that change data are marked as such, and AI apps ask for your confirmation before running them. Read what the app is about to do before you approve it.
- Want a read-only connection? Create a dashboard user with a read-only role and connect with that user.
- A connection lasts as long as a Getgud login, 30 days. After that the AI app asks you to log in again.

## What you can ask

**Investigate players and matches**

- "How many insights did we get in Title X this week, split by toxic behavior?"
- "Show me the 10 most toxic players in the last 24 hours and what they were flagged for."
- "Which matches yesterday had more than 3 reports?"
- "Did aimbot detections go up after Tuesday's patch? Show me the daily trend."

**Analyze game balance**

- "Which weapons are overperforming in Title X? Look at usage, kill share and accuracy."
- "Compare the win/loss ratio of our characters on each map."
- "What is the time to kill of the new rifle compared with the old one?"

**Understand a player**

- "Why was this player flagged for rapid fire? Look at their player model."
- "List every report filed against this player and who filed them."

**Manage your setup**

- "Turn on the AFK guard law for our QA title."
- "Create a rule that sends players with a toxicity score above 80 to our webhook."
- "Save this search as a query and pin it to my dashboard."
- "Invite a new teammate with the analyst role and give them access to Title X."

The server ships with its own documentation for the AI: what the numbers mean, the formulas behind the Analyze screen, the names behind every id. Your AI app reads it on its own when it needs to, so you can ask in your own words.

## Tools

| Area | Tools |
|------|-------|
| Search | `list_titles`, `find_matches`, `find_players`, `find_insights`, `find_reports` |
| Matches and maps | `get_match`, `get_match_action_stream`, `get_match_chat`, `get_live_games_info`, `get_live_game_packets`, `get_map_data`, `upload_client_map`, `save_games` |
| Models | `get_title_model`, `update_title_model`, `get_player_model` |
| Players and reports | `update_players`, `delete_players`, `send_reports` |
| Guard laws, filters, rules | `update_guard_laws`, `remove_guard_laws`, `update_filters`, `delete_filters`, `update_rules`, `delete_rules` |
| Titles | `create_title`, `update_title`, `delete_titles`, `delete_titles_maps`, `delete_titles_assets`, `reset_title_private_key`, `update_npc_settings`, `update_affect_mapping`, `update_heatmaps_state`, `update_asset_variants` |
| Queries and AI reports | `get_saved_queries_info`, `save_queries`, `delete_queries`, `update_dashboard_queries`, `update_ai_reports`, `delete_ai_reports` |
| Organization and users | `get_client_info`, `get_users_info`, `get_user_roles`, `create_user`, `change_user_role_to_users`, `update_user_roles`, `delete_user_roles`, `assign_users_to_titles`, `unassign_users_to_titles`, `delete_users` |
| Documentation for the AI | `get_guide` (analyze, match, investigate, glossary), `get_decoding_guide` |

Sending game data is not part of the MCP. That is the job of the [Getgud SDK](https://github.com/getgud-io/getgud-docs/blob/main/get-started.md).

## Working with match data

Getgud stores every match as an **action stream**: a compact record of every position, spawn, attack, damage, death, heal, affect and custom event. A single match can hold millions of actions, so the MCP server hands the stream over exactly as stored (packed) and your AI app decodes it on your machine.

The server teaches the AI how. `get_decoding_guide` returns the format and a ready-made decoder for the machine it runs on, using only what ships with the operating system:

| System | Decoder |
|--------|---------|
| macOS, Linux | Perl |
| Windows | Windows PowerShell |
| Any system with Node.js or Python, and AI chat code sandboxes | Node.js, Python |

This works best in AI coding tools that can run commands on your machine, such as Claude Code, Cursor and Codex: ask "export match 12345 to JSON" or "who killed whom in match 12345" and the tool downloads, decodes and answers. Chat apps without access to your machine can answer everything else, but cannot decode large matches. For those, use **Export Matches** in the dashboard's Investigate screen and upload the file to your chat.

## Security and privacy

- **No stored data.** The MCP server has no database. It keeps none of your Getgud data and none of your login.
- **Your login stays sealed.** After you log in, the server encrypts your session with a key only it holds and gives the sealed result to your AI app. The AI app sends it back with each request and cannot read it. Your password is never sent to the MCP server or the AI app: it goes from the Getgud dashboard straight to our login service.
- **Separate from your dashboard session.** Signing out of the dashboard does not disconnect your AI app, and the other way around.
- **Disconnect any time** by removing the Getgud connector in your AI app.
- **Text written by players reaches your AI app.** Nicknames, chat messages and report text are written by players. The server tells the AI to treat them as data, never as instructions. Still, review what your AI app asks to change before you approve it, above all deletions.

## FAQ

**Can I limit what the AI app can do?**
Yes. It can never do more than the user you connect with. Connect with a user whose role has only the permissions you are comfortable with.

**The AI app says the login expired.**
Connections last 30 days. Reconnect from your AI app's connector settings and log in again.

**I get "not found" for a match.**
Match data is kept for your plan's retention period. Matches that were purged return not found. Mark games as saved (or ask the AI to) to keep them longer.

**Something is not working.**
Remove the connector, add it again and log in. If it still fails, contact us with the name of your AI app and the time of the attempt.
