Define a **GMAIL** operation step in JSON workflow language to connect to Gmail (Google Workspace) and fetch emails from a mailbox using a **pipeline connector** (`db_org_pipeline_connector`).

This operation uses a **service account JSON key** with **domain-wide delegation**. The connector must store:
- `config.email`: the mailbox to impersonate (e.g. `user@company.com`)
- `credentials.service_account_key`: the full JSON key contents (paste the entire key file)

**OAuth pipeline connector (not in use today):** A user-consent / authorization-code OAuth flow for Gmail (and Google Drive) pipeline connectors is **not** wired in the current executor or product path; production paths use **service account + domain-wide delegation** as above. Spec text, credential shapes, and related notes for an OAuth-style connector are kept in the Gmail and Google Drive operation-description markdown files in this folder (`GMAIL_OPERATION_DESCRIPTION.md`, `GOOGLE_DRIVE_READ_OPERATION_DESCRIPTION.md`, `GOOGLE_DRIVE_WRITE_OPERATION_DESCRIPTION.md`) for reference when that flow is enabled.

Minimal example:

```json
{
  "id": "fetch_latest_emails",
  "operation": "GMAIL",
  "connector_id": "<pipeline_connector_id>",
  "fetch_mode": "LAST_N_EMAILS",
  "last_n_mails": 2,
  "labels": "INBOX",
  "include_attachments": false,
  "select_fields": ["mail_id", "subject", "from", "received_datetime", "mail_content", "adopt_internal_source_id"],
  "notes": "Fetch the latest 2 emails from Gmail."
}
```

### Required fields
- `id` (string): unique step id
- `operation` (string): must be `"GMAIL"`
- `connector_id` (string): id of the saved pipeline connector containing the service account key
- `notes` (string): short description of what the step does

### Optional fields
- `fetch_mode`: only `"LAST_N_EMAILS"` is supported
- `last_n_mails` (int): how many messages to return
- `labels` (string): comma-separated label names/ids (e.g. `"INBOX,SENT"`). Leave empty for all.
- `include_attachments` (bool): when true, include attachment metadata when available (best-effort)
- `select_fields` (string[]): if provided, project each email to just these fields (but always keep `adopt_internal_source_id`)

### Output shape
**CRITICAL — same as OUTLOOK:** GMAIL returns a **JSON object** (not a bare array). The messages are in the `emails` key. Downstream `JQ_FILTER` steps that extract the list should use filter `.emails` with `extract_all: false` (see `JQ_FILTER_OPERATION_DESCRIPTION.md`). Using `.emails` on a bare array causes jq error: `Cannot index array with string "emails"`.

Top-level fields:
- `emails`: **array** of email objects (see below)
- `total_emails_fetched` (integer)
- `fetch_mode` (string, e.g. `LAST_N_EMAILS`)
- `selected_fields` (string[], optional): present when `select_fields` was set on the step

Each object in `emails` includes:
- `mail_id` (string)
- `subject` (string)
- `from` (string)
- `to_recipients` (string)
- `received_datetime` (string, ISO)
- `sent_datetime` (string, ISO)
- `mail_content` (string, best-effort plain text)
- `unique_body` (string, alias for mail_content)
- `snippet` (string)
- `adopt_internal_source_id` (string): a stable per-email lineage key

### Writing to DB
Unlike `OUTLOOK`, `GMAIL` does **not** auto-write emails to a table. If the user asks to persist raw emails, add an explicit `WRITE_TO_DB` step immediately after `GMAIL` with the GMAIL step id as input. `WRITE_TO_DB` unwraps the `emails` array from this object automatically (same as for OUTLOOK).

### FAN_OUT
Do **not** wire `FAN_OUT` `input` directly to a GMAIL step id. Add a `JQ_FILTER` with `.emails` (and `extract_all: false`) first, then point `FAN_OUT` at that filter step — same pattern as OUTLOOK.

