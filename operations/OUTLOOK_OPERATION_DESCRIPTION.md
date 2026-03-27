Define an OUTLOOK operation step in JSON workflow language to connect to Outlook (Microsoft 365) and fetch emails from a mailbox.

Basic Structure:
{
  "id": string,
  "operation": "OUTLOOK",
  "connector_id": string,
  "table_id": string,
  "table_label": string,
  "sync_mode": string,
  "only_new": boolean,
  "write": boolean,
  "fetch_mode": string,
  "lookback_minutes": integer,
  "last_n_mails": integer,
  "start_time": string,
  "end_time": string,
  "from_time": string,
  "to_time": string,
  "limit": integer,
  "page_size": integer,
  "max_pages": integer,
  "select_fields": list[string],
  "fields": list[string]
}

Supported fetch_mode values:
- LAST_N_MINUTES: Fetch emails from the last `lookback_minutes` (default 10).
- LAST_YEAR: Fetch emails from the last 365 days.
- LAST_N_EMAILS: Fetch the latest `last_n_mails` emails (default 100).
- TIME_RANGE: Fetch emails between `start_time`/`from_time` and `end_time`/`to_time`.

Supported sync_mode values:
- ALL: Return all emails that match the fetch window.
- ONLY_NEW: If target table exists and has mail `received_datetime`, return only newer emails than the latest stored one.

Notes:
- The operation automatically paginates over Microsoft Graph `@odata.nextLink` responses to handle large result sets.
- Authentication is resolved using `connector_id` via `IntegrationToolDB.get_authentication_spec(org_id, connector_id)`.
- If `table_id` or `table_label` is provided, table lookup is attempted from pipeline registry.
- If the resolved table does not exist, table-aware filtering/writing is skipped gracefully.
- If `write=true` and table exists, fetched emails are also written to that table.
- Use `select_fields` (or `fields`) for selective data fetching.
- Common selectable fields include: `mail_id`, `subject`, `mail_content`, `unique_body`, `from`, `from_name`, `to_recipients`, `cc_recipients`, `bcc_recipients`, `received_datetime`, `sent_datetime`, `is_read`, `importance`, `has_attachments`, `web_link`, `internet_message_id`, `conversation_id`.
- Default to `unique_body` over `mail_content` unless the user explicitly requests full thread content. `unique_body` returns only the new content added in that specific message (Microsoft Graph `uniqueBody`), stripping all quoted history — this avoids bloated payloads from reply chains. Only use `mail_content` when the user specifically asks for the full body or complete thread history.

Output Format:
CRITICAL — OUTLOOK returns a dict object (NOT an array). The emails list is inside the "emails" key. Any downstream step that needs the array of email rows (e.g. FAN_OUT, JQ_FILTER expecting a list) MUST have a JQ_FILTER step immediately after OUTLOOK with filter ".emails" and extract_all false. NEVER wire FAN_OUT "input" directly to an OUTLOOK step id — always route through that JQ_FILTER.

The OUTLOOK step returns an object with the following fields:
- `emails`: list[dict] — Fetched mail objects (each containing the selected fields).
- `total_emails_fetched`: integer — Number of emails returned.
- `pages_fetched`: integer — Number of Microsoft Graph API pages fetched.
- `fetch_mode`: string — The fetch mode used (e.g. LAST_N_MINUTES, LAST_N_EMAILS, etc.).
- `selected_fields`: list[string] — Fields projected in the response.
- `start_time_utc`: string | null — Fetch window start time in UTC (ISO 8601).
- `end_time_utc`: string | null — Fetch window end time in UTC (ISO 8601).
- `has_more`: boolean — True if more pages existed but were not fetched (e.g. due to max_pages cap).
- `sync_mode`: string — Sync mode used (ALL or ONLY_NEW).
- `table_name`: string | null — Resolved physical table name.
- `table_exists`: boolean — Whether the target table existed at execution time.
- `latest_existing_received_datetime`: string | null — Latest `received_datetime` found in the target table before fetch (used by ONLY_NEW sync).
- `write_requested`: boolean — Whether `write=true` was set.
- `rows_written`: integer — Number of rows written to the target table.

Example Output:
{
  "emails": [
    {
      "mail_id": "AAMkADBhNzFj...",
      "subject": "Shipping Again Soon? We are Ready to Help at the Best Price!",
      "received_datetime": "2026-03-05T16:18:12Z",
      "mail_content": "<html>...</html>"
    }
  ],
  "total_emails_fetched": 1,
  "pages_fetched": 1,
  "fetch_mode": "LAST_N_MINUTES",
  "selected_fields": ["mail_id", "subject", "received_datetime", "mail_content"],
  "start_time_utc": "2026-03-05T15:37:20Z",
  "end_time_utc": "2026-03-05T16:22:14Z",
  "has_more": false,
  "sync_mode": "ONLY_NEW",
  "table_name": "pipeline_random006_93a05242be734410_test",
  "table_exists": true,
  "latest_existing_received_datetime": "2026-03-05T15:37:20Z",
  "write_requested": true,
  "rows_written": 1
}

Test Mode Behaviour:
- When `test_mode=true`, mail fetching is capped at `test_mode_max_mails` (default 3).
- If 0 mails are fetched under test_mode, execution stops immediately with an error indicating that no mails matched the given filters, and further nodes are not executed.

Generate these keys but leave values empty strings unless the workflow already has concrete values.
