Define a PARSE_DOCUMENT operation step in a JSON workflow language that downloads and parses file(s) from S3, SharePoint, or Google Drive for use by downstream PROMPT or AGENTIC_SEARCH steps.

Basic Structure:
{
  "id": string,
  "operation": "PARSE_DOCUMENT",
  "connector_id": string (pipeline connector — S3, SharePoint, or google_drive),
  "source_type": string (optional; "s3", "sharepoint", or "google_drive"; default "s3"),
  "file_path": string (see File Path Rules; for google_drive see Google Drive section below),
  "folder_path": string (optional; for SharePoint, the base folder within the drive),
  "site_id": string (optional; for SharePoint, overrides connector-configured site),
  "drive_id": string (optional; for SharePoint, overrides connector-configured drive),
  "site_url": string (optional; for SharePoint, used to resolve site_id if absent),
  "user_email": string (optional; for google_drive, overrides delegated user),
  "drive_resource_url": string (optional; for google_drive, overrides connector link),
  "drive_resource_id": string (optional; for google_drive, overrides stored file/folder id),
  "batch_limit_files": number (optional; for google_drive, max files per run),
  "batch_limit_size_bytes": number (optional; for google_drive, total download cap),
  "table_label": string (optional; for google_drive only — logical name for the pipeline table where parsed rows are stored),
  "test_mode_max_files": number (optional; when executor test_mode is true, caps wildcard/listing for s3/sharepoint; for google_drive passed as fetch cap)
}

Google Drive (source_type "google_drive"):
- Uses the same resolution, download, and text extraction as GOOGLE_DRIVE_READ (Drive API by file id; Google Sheets exported as CSV via files.export; ingestion rules match that operation).
- Returns parsed document content for downstream workflow steps, but **does not automatically persist parsed rows to a pipeline table** in the current runtime behavior.
- **`table_label`**: optional metadata for Google Drive steps, but it is **not currently used to auto-create or populate a pipeline table** during `PARSE_DOCUMENT` execution. Do not rely on downstream EMBEDDER / READ_FROM_DB being able to reference a Drive parse result by `table_label` unless that persistence is added separately.
- **`table_label`**: optional. If the user does not name a table, the runtime derives a stable label from the step **`id`** (sanitized, max 64 chars), e.g. step `id` `parse_documents` → label `parse_documents`. Prefer an explicit `table_label` when the user names a destination or when multiple Drive parse steps need distinct tables.
- connector_id must reference a google_drive pipeline connector.
- **What to put in `file_path`:** Google Drive is not a bucket with prefixes like S3. The **authoritative target is the Drive file or folder link (or id) stored on the connector** when the user creates the connection—the same value as GOOGLE_DRIVE_READ’s `file_or_folder_path` in connector config. For most pipelines, **omit `file_path`** so the step uses that stored link. To override for one step only, set `file_path` to another **Drive URL**, **raw file/folder id**, or legacy slash path—**not** a glob. The S3/SharePoint wildcard `*` does **not** mean “all files in Drive”; it is only a UI/legacy placeholder and the runtime **ignores `*` and `**` for Drive**, falling back to the connector’s configured link/path/id.
- Optional step fields `drive_resource_url` and `drive_resource_id` override the connector’s link/id when you need an explicit anchor without putting it in `file_path`.

File Path Rules (S3 / SharePoint only — not Google Drive):
- file_path is RELATIVE to the connector's root path (S3 prefix for S3, folder_path for SharePoint). Do NOT repeat the bucket name or the connector prefix in file_path — it is automatically prepended.
  Example (S3): if BucketURI = "s3://my-bucket/org_data/docs/" and the file is at s3://my-bucket/org_data/docs/report.pdf, then file_path = "report.pdf" (NOT "org_data/docs/report.pdf").
- When the user does NOT specify a particular file name, use file_path = "*" to process ALL supported files in the connector's path. This is the most common pattern for pipelines wired to an S3 or SharePoint source. **Do not apply this mental model to `source_type` `google_drive`**—see the Google Drive section above.
- Glob patterns are supported: "*.pdf" (all PDFs), "reports/*.csv" (CSVs in reports/ subfolder).
- Template variables are supported: "{{previousStep.relative_path}}" resolves at runtime.

Output (single file — file_path is a concrete path, or google_drive returned one file):
{
  "text": string,
  "pages": [{"page_number": int, "text": string}, ...],
  "filename": string,
  "file_type": "pdf" | "csv" | "txt" | "json" | "md" | "xml" | "html",
  "file_path": string,
  "s3_url": string (S3 only, when applicable),
  "google_drive_file_id": string (google_drive only, when applicable),
  "page_count": int,
  "adopt_internal_source_id": string (when provided by source)
}

Output (wildcard / glob for S3/SharePoint — file_path contains * or ? — or multiple files from a **Google Drive folder** listing):
{
  "documents": [
    {"text": string, "pages": [...], "filename": string, "file_type": string, "file_path": string, ...},
    ...
  ],
  "total_files": int,
  "failed_files": int,
  "file_path": string (the original pattern)
}

Key Features:
- Downloads or reads using pipeline connector credentials
- Supports PDF (extracts pages with page numbers where available), text, CSV, JSON, XML, HTML
- source_type specifies the backend: "s3" (default), "sharepoint", or "google_drive"
- Wildcard mode processes all matching files and returns a documents array

Implementation Notes:
- connector_id is required; org_id is injected from the executor context; file_path is required for s3 and sharepoint; for google_drive it is optional when the connector already has the pasted link/id
- Credentials are resolved from db_org_pipeline_connector via the connector_id
- If text extraction fails or returns empty, an error is returned
- When using wildcard, partially failed files are skipped; the operation succeeds if at least one file parses
