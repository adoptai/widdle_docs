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
  "fields": list[string],
  "download_attachments": boolean,
  "s3_connector_id": string | null,          // only required when download_attachments=true
  "max_attachment_size_bytes": integer,       // default -1; only when download_attachments=true; -1 = unlimited
  "allowed_extensions": list[string] | null   // only applicable when download_attachments=true; null = accept all
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
- `total_attachments_downloaded`: integer — Count of attachments successfully downloaded from Graph and uploaded to S3 (0 unless `download_attachments=true`).
- `total_attachment_errors`: integer — Count of attachments that failed to download or upload.
- `uploaded_attachment_paths`: list[string] — Relative S3 paths (relative to the S3 connector's `BucketURI` prefix) of successfully uploaded attachments. Wire this into a downstream `PARSE_DOCUMENT` step's `file_path` for incremental parsing.
- `uploaded_attachment_download_urls`: list[string] — Presigned HTTPS download URLs (TTL: 6 hours) for successfully uploaded attachments. Wire this into sandbox/custom-extractor manifests that consume `url`.
- `uploaded_attachment_manifest`: list[dict] — Per-attachment records with `mail_id`, `attachment_id`, `filename`, `relative_path`, `download_url`. Primary field for mail→file mapping in downstream pipelines.
- `total_bytes_uploaded`: integer — Total bytes uploaded to S3 across all attachments.
- `upload_duration_seconds`: number — Cumulative wall-clock time spent on S3 PUTs for attachments.
- `skipped_by_size_filter`: integer — Attachments skipped because they exceeded `max_attachment_size_bytes`.
- `skipped_by_extension_filter`: integer — Attachments skipped because their extension was not in `allowed_extensions`.
- Each email object may also include `attachment_count` (integer) and `attachment_filenames` (list[string]) when `download_attachments=true`.

Attachment Download (v1):
- Enable by setting `download_attachments=true` and providing `s3_connector_id` (the connector that holds the AWS credentials and `BucketURI`). Both are required together; validation fails otherwise.
- Storage layout: `s3://<bucket>/<BucketURI-prefix>/pipeline_{pipeline_id}/attachments/dt={YYYY-MM-DD}/mail_{mail_short}/{att_short}_{filename}`. The prefix comes from the S3 connector's `BucketURI`; we never hardcode `org_{org_id}` into the key. `mail_short` and `att_short` are 10-char SHA-1 hex digests of the raw Graph mail/attachment ids — short and stable, so the same email always lands in the same partition.
- Filename is sanitized (non-alphanumeric chars become `_`) so Graph-issued attachment names produce valid S3 keys; raw mail/attachment ids are hashed (not sanitized inline) for readability.
- For downstream consumers that need an explicit mail→file mapping without parsing the path, use the `uploaded_attachment_manifest` output: a list of `{mail_id, attachment_id, filename, relative_path, download_url}`.
- Filters:
  - `max_attachment_size_bytes` (default `-1`, unlimited): attachments with `size` > this value are skipped and counted in `skipped_by_size_filter`.
  - `allowed_extensions` (default `null`, accept all): case-insensitive, leading dot optional, e.g. `["pdf", "xlsx", "docx"]`. Non-matching files are counted in `skipped_by_extension_filter`.
- v1 supports Graph file attachments (`microsoft.graph.fileAttachment`) only. Item attachments (forwarded emails, calendar items) and reference attachments (OneDrive links) are skipped.
- S3 upload retries 429/503/504 responses with exponential backoff (1s, 2s, 4s, capped at 30s). Permanent failures increment `total_attachment_errors` and the email continues processing.
- Use `uploaded_attachment_paths` with `PARSE_DOCUMENT` (relative path expected) and `uploaded_attachment_download_urls` with sandbox / custom extractors (full presigned HTTPS URLs, valid for 6 hours).
- For hourly pipelines that must avoid re-emitting old attachments, combine `download_attachments=true` with `sync_mode=ONLY_NEW`, a resolved `table_id`/`table_label`, and `write=true` so the mail-level watermark advances each run.

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
