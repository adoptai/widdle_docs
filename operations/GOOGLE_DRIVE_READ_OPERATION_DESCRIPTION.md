Define a GOOGLE_DRIVE_READ operation step in a JSON workflow language that resolves a path in Google Drive (My Drive or a shared drive), downloads matching files, extracts text from txt/csv/pdf, and optionally writes rows to a pipeline table for EMBEDDER (table source).

Basic Structure:
{
  "id": string,
  "operation": "GOOGLE_DRIVE_READ",
  "connector_id": string (reference to google_drive pipeline connector),
  "write": boolean (optional, default true — persist rows to pipeline table),
  "table_label": string (required when write is true; unique vs other S3_READ / GOOGLE_DRIVE_READ / EMBEDDER labels),
  "table_id": string (optional; registry id — usually omit and use table_label only),
  "batch_limit_files": number (optional, default 100),
  "batch_limit_size_bytes": number (optional, default 500MB total download cap),
  "file_or_folder_path": string (optional override; otherwise connector config path),
  "user_email": string (optional override for domain-wide delegation subject)
}
Note: Path, drive_id, extensions, recursive, and service account key normally come from the connector (config + encrypted credentials). org_id is injected by the executor — do NOT put org_id in the step.

**OAuth pipeline connector (not in use today):** A user-consent / authorization-code OAuth flow for Google Drive (and Gmail) pipeline connectors is **not** wired in the current executor; runtime auth is **service account + domain-wide delegation** as in the implementation notes below. Documentation for an OAuth-oriented connector lives alongside this spec in `GMAIL_OPERATION_DESCRIPTION.md` and the other Google Drive operation-description files in `actionbot/explanation_prompt_md_collection/`.

Key Features:
- Resolves slash-separated paths via Drive API queries (not POSIX); fails if a segment matches multiple names.
- Shared drive: connector drive_id selects corpora=drive listings.
- Supports native Google Sheets by exporting them as CSV; skips other native Google Apps files (for example Docs/Slides) for download.
- Returns a dict when write is true: `{ "files": [...], "file_count": N, "errors": [...] }` — use JQ_FILTER `.files` before FAN_OUT.
- When write is false, returns a list of row objects directly.

Typical pattern:
1. GOOGLE_DRIVE_READ with write:true and table_label (e.g. "gdrive_docs").
2. EMBEDDER with input_source_type "table" and input_source_info.table_label matching that label.

Implementation Notes:
- adopt_internal_source_id is a stable 32-char hex derived from the Drive file id (fits source_id column width).
- Each row includes body/content text plus filename, gdrive_file_id, mime_type for downstream use.
- Requires domain-wide delegation: service account impersonates user_email from connector config.
