# CourtAPI plugin

Your AI docketing assistant for US federal courts. This plugin connects your agent to
[CourtAPI](https://www.courtapi.com) through the hosted CourtAPI MCP server. Use it to find
cases, read docket sheets, claims registers, creditors, parties and documents, follow your
firm's recent case activity and CourtDrive calendar, and refresh records from PACER when you
choose to pay for that.

The same package works in several clients:

| Client | Manifest | MCP config |
|---|---|---|
| [Agent Plugins](https://agent-plugins.org) clients, including Microsoft Copilot through WIQD | `plugin.json` | `mcp.json` |
| Grok Build | `.grok-plugin/plugin.json` | `.mcp.json` |
| Claude Code | `.claude-plugin/plugin.json` | `.mcp.json` |

## Installation

- **Grok Build:** open `/plugin`, search for **CourtAPI**, and install.
- **Claude Code:** `claude plugin marketplace add courtio/courtapi-plugin`, then
  `claude plugin install courtapi@courtapi`.
- **Any other MCP client:** add the remote server `https://mcp.courtapi.com/mcp` (Streamable HTTP).

The first time you connect, your client opens the CourtAPI sign-in page in your browser. Enter
the App ID and App Key of your CourtAPI application, from your
[CourtAPI account](https://www.courtapi.com). Your client never sees the key. Nothing should
be pasted into chat.

## What you get

- **MCP server** `courtapi` at `https://mcp.courtapi.com/mcp` (Streamable HTTP, OAuth 2.0 with
  PKCE and dynamic client registration).
- **Skill** `courtapi`: how to use the tools well. It covers reading before refreshing,
  confirming billable PACER calls, the case → docket → document flow, and async refreshes.

### Tools

| Area | Free reads | Billable (asks first) |
|---|---|---|
| Find cases | `find_cases`, `list_courts` | `ncl_search` |
| Case | `get_case` | `refresh_case_from_pacer` |
| Docket | `get_dockets` | `refresh_dockets_from_pacer` |
| Documents | `get_document`, `fetch_document_text` | `purchase_document_from_pacer` |
| Claims | `get_claims` | `refresh_claims_from_pacer` |
| Creditors | `get_creditors` | `refresh_creditors_from_pacer` |
| Parties | `get_parties` | |
| Firm | `get_case_activity` | `get_calendar_events` |
| PACER | `check_pacer_credentials`, `get_pacer_refresh_status` | |

Every tool carries MCP annotations (`readOnlyHint`, `destructiveHint`, `openWorldHint`).
Each billable tool is marked `destructiveHint: true`, and its title says so, so your client
confirms before spending money.

## Example prompts

- "Find the Sbarro bankruptcy and summarize the last ten docket entries."
- "Who filed the five largest unsecured claims in that case?"
- "List the attorneys for each party."
- "What's new on our firm's tracked cases this week?"
- "What hearings do I have on my calendar next week?"
- "Is docket entry 42 cached? If not, how much would it cost to pull from PACER?"

## Authentication, data and network

The plugin connects only to `https://mcp.courtapi.com`. It requests no filesystem or shell
access and ships no hooks or scripts.

Network endpoints:

- `https://mcp.courtapi.com/mcp`: the hosted MCP server (Streamable HTTP)
- `https://mcp.courtapi.com/register`, `/authorize`, `/connect`, `/token`: OAuth 2.0 with
  dynamic client registration and PKCE
- `https://mcp.courtapi.com/.well-known/oauth-authorization-server` and
  `/.well-known/oauth-protected-resource`: discovery metadata

Credentials: a CourtAPI App ID and App Key, entered once on the CourtAPI sign-in page. The
server binds them to the OAuth token it issues. Tools act only on data that CourtAPI
application can already reach. Billable PACER calls also need PACER credentials stored in
your CourtAPI account. The MCP server never accepts or returns PACER passwords.

Costs: billable tools charge your CourtAPI account, and PACER's own fees apply. Check usage on
the [CourtAPI usage dashboard](https://courtapi.com/buyer/stats).

## Support and policies

- Support: [support@courtapi.com](mailto:support@courtapi.com)
- Privacy policy: <https://www.courtapi.com/legal/privacy>
- Terms of service: <https://www.courtapi.com/legal/tos>

## License

The files in this plugin are MIT licensed (see [LICENSE](LICENSE)). Use of the hosted MCP
server and CourtAPI data is governed by the CourtAPI terms of service and privacy policy.
