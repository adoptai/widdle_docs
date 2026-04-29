Define a GOOGLE_DRIVE_WRITE operation step in a JSON workflow language. Uploads file content to Google Drive or creates a folder under a configured parent path, using the same pipeline Google Drive connector (service account + domain-wide delegation) as GOOGLE_DRIVE_READ.

**OAuth pipeline connector (not in use today):** A user-consent / authorization-code OAuth flow for Google Drive (and Gmail) pipeline connectors is **not** wired in the current executor. Retained spec and examples for that approach are in the Gmail and Google Drive operation-description markdown files in `actionbot/explanation_prompt_md_collection/` (this file, `GOOGLE_DRIVE_READ_OPERATION_DESCRIPTION.md`, and `GMAIL_OPERATION_DESCRIPTION.md`).

**Workspace admin (domain-wide delegation):** The service account must be granted the Google Drive API scope `https://www.googleapis.com/auth/drive` in Google Workspace Admin (domain-wide delegation) in addition to any existing `drive.readonly` scope used for reads. Without this scope, token exchange or uploads will fail.

Basic Structure:
{
  "id": string,
  "operation": "GOOGLE_DRIVE_WRITE",
  "connector_id": string (google_drive pipeline connector id),
  "input": string (step id whose output becomes file bytes/text/JSON — required when item_type is file or omitted),
  "item_type": string (optional; "file" default, or "folder" to create a single folder),
  "file_name": string (optional; default export.json or export.txt from mime),
  "folder_name": string (required for item_type folder — name of the new folder),
  "folder_path_to_upload": string (optional slash path for parent folder; else connector config `folder_path_to_upload`),
  "mime_type": string (optional content type for upload),
  "user_email": string (optional override for delegated user),
  "create_parent_folders": boolean (optional, default true — create missing segments in folder_path_to_upload),
  "notes": string
}

Key Behavior:
- **File upload:** Resolves parent folder from `folder_path_to_upload` (step or connector). When `create_parent_folders` is true, missing folder segments are created. Serializes prior-step output: strings and bytes as-is; dict/list as JSON. Uses multipart upload to Drive API v3.
- **Folder create:** Creates one folder named `folder_name` (or `file_name` if folder_name omitted) under the resolved parent path.
- **Top-level only:** Do not place GOOGLE_DRIVE_WRITE inside FAN_OUT `sub_steps` — use a top-level step after aggregation (e.g. after JQ_FILTER or PROMPT).

Typical pattern:
1. PROMPT or JQ_FILTER produces structured or text output.
2. GOOGLE_DRIVE_WRITE with `input` set to that step's id, `file_name`, optional `folder_path_to_upload` matching connector "Upload Folder Path".

Pipeline pattern when destination is Google Drive:
  … → GOOGLE_DRIVE_WRITE (connector_id from destinations / same Drive connector) → END
