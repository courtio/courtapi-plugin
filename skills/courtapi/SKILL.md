---
name: courtapi
description: >-
  Work with US federal court records through the CourtAPI MCP server: find cases,
  read case details, docket sheets, bankruptcy claims registers, creditor lists,
  parties and attorneys, open or read court documents, follow the firm's recent
  case activity and CourtDrive calendar, and refresh records from PACER when the
  user chooses to pay for it. Use whenever CourtAPI, CourtDrive, a federal court
  case, docket, bankruptcy filing, proof of claim, or a court_code/case_number
  pair comes up.
license: MIT
---

# CourtAPI

CourtAPI keeps a copy of US federal court records (bankruptcy, district and
appellate) from PACER. The `courtapi` MCP server bundled with this plugin acts as
the signed-in CourtAPI app: its reads come from CourtAPI's stored copy, and a few
tools reach PACER live for a fee.

## Free reads and billable PACER calls

Most tools read CourtAPI's stored copy. They are instant and free:
`find_cases`, `get_case`, `get_dockets`, `get_document`, `get_claims`,
`get_creditors`, `get_parties`, `get_case_activity`, `list_courts`,
`check_pacer_credentials`, `get_pacer_refresh_status`, `fetch_document_text`.

These tools cost money, and their titles say so. Each one carries
`destructiveHint: true` so the client asks before calling it:

| Tool | What it costs |
|---|---|
| `refresh_case_from_pacer`, `refresh_dockets_from_pacer`, `refresh_claims_from_pacer`, `refresh_creditors_from_pacer` | A live PACER fetch, billed by PACER |
| `purchase_document_from_pacer` | The document's PACER price (`download_cost` from `get_document`); free if it turns out to be cached already |
| `ncl_search` | About $0.10 per National Case Locator result page |
| `get_calendar_events` | One CourtAPI call per page, plus a one-time CourtDrive charge per newly delivered upcoming event |

Rules:

1. **Read before you refresh.** Always try the free read first. Refresh only when
   the stored copy is missing what the user needs, or they ask for the latest
   from PACER.
2. **Say what it will cost and get a yes** before any billable call, unless the
   user has already asked for that exact action. Quote `download_cost` for
   documents.
3. **Narrow refreshes.** Pass `date_filed_from` to `refresh_dockets_from_pacer`
   and `refresh_claims_from_pacer` so they fetch only new entries.
4. Billable PACER calls need stored PACER credentials. If
   `check_pacer_credentials` returns `configured: false`, tell the user to add
   them on their CourtAPI account settings page. Never ask for a PACER password
   in chat.

## Typical flow

1. **Find the case.** `find_cases` with a party name, partial case number or
   keywords returns `court_code` + `case_number`, which every other tool needs.
   If you need a court code, use `list_courts`. Use `ncl_search` (billable) only
   for cross-court party searches that `find_cases` cannot answer.
2. **Get the case.** `get_case` returns metadata and a `menu` of which
   sub-resources exist. Call it first.
3. **Read what you need.** `get_dockets` (supports `search_keyword`, date and
   docket-number ranges), `get_claims`, `get_creditors` (scheduled creditors,
   unlike `get_claims`, which lists filed proofs of claim), `get_parties` (pass
   `include_attorneys` for counsel and contact details).
4. **Documents.** Take `docket_seq` from a docket entry (for example
   `42.00000`) and call `get_document`.
   - If the user wants the document, give them the signed `download_url` or
     `preview_url`. It opens in their browser.
   - If you need to read it yourself, pass its `ocr_link` to
     `fetch_document_text`.
   - If it is not cached, quote `download_cost` and buy it with
     `purchase_document_from_pacer` only after a yes.
5. **Async refreshes.** `refresh_dockets_from_pacer` and
   `refresh_claims_from_pacer` return a `task_id`. Poll
   `get_pacer_refresh_status` every few seconds until it is no longer `pending`,
   then read again with `get_dockets` or `get_claims`.

## Firm features

- `get_case_activity` shows new dockets and claims on the firm's tracked cases,
  for the last 3 days by default (widen with `since_days`). It returns nothing if
  the CourtAPI app is not linked to a firm.
- `get_calendar_events` reads hearings and deadlines from the linked CourtDrive
  user's calendar. It needs `date_from` and `date_to`, and it needs the AI
  Calendar permission. Keep the date range tight: this tool is billable, and it
  is not meant for polling.

## Good practice

- Never guess a `court_code`, `case_number` or `docket_seq`. Look it up.
- Cite the docket number and filing date when you summarize filings.
- For questions about CourtAPI usage, billing or remaining credits, point the
  user to the [CourtAPI usage dashboard](https://courtapi.com/buyer/stats).
- CourtAPI provides court records, not legal advice. Do not present your summary
  as a substitute for reading the filing when the stakes are high.
