Define a SHAREPOINT_WRITE operation step in a JSON workflow language. Uploads file content produced by a prior step to a folder in a Microsoft SharePoint document library, using the same pipeline SharePoint connector (Azure AD app, app-only client credentials) as SHAREPOINT_READ. It is the symmetric counterpart to SHAREPOINT_READ.

**App permission (write scope required):** The connector's Azure AD app must hold **write** access to the target site — `Sites.Selected` granted with the `write` role on that site (in addition to the read role used by SHAREPOINT_READ). Without it, Microsoft Graph returns `403 accessDenied` and the upload fails.

**Site/library targeting:** Site and drive (document library) are resolved from the connector configuration exactly as SHAREPOINT_READ resolves them (including `library_name` when the target is a non-default library). The step's `folder_path` is the drive-relative destination folder within that library.

Basic Structure:
{
  "id": string,
  "operation": "SHAREPOINT_WRITE",
  "connector_id": string (sharepoint pipeline connector id),
  "input": string (step id whose output becomes the file bytes/text/JSON — required),
  "file_name": string (optional; default export.json or export.txt from mime),
  "folder_path": string (optional drive-relative destination folder; "" == library root; also accepts "sharepoint_folder_path"; else connector config folder_path),
  "sharepoint_folder_path": string (optional alias for folder_path),
  "mime_type": string (optional content type for the uploaded file),
  "overwrite": boolean (optional, default true — replace an existing file at the path; SharePoint retains version history),
  "notes": string
}

Key Behavior:
- **File upload:** Resolves the target drive from the connector, then uploads `input`'s output to `{folder_path}/{file_name}`. Serializes prior-step output: strings and bytes as-is; dict/list as JSON.
- **Small vs large:** Files ≤ 4 MB use a single `PUT .../root:/{path}:/content`; larger files use a Graph upload session (`createUploadSession`) with chunked `PUT` (chunk size a multiple of 320 KiB), so large deliverables (e.g. RTC Model workbooks) upload reliably.
- **Overwrite:** With `overwrite` true (default) an existing file at the path is replaced via `@microsoft.graph.conflictBehavior=replace`; SharePoint keeps prior versions. Set false to fail on conflict.
- **Top-level only:** Do not place SHAREPOINT_WRITE inside FAN_OUT `sub_steps` — use a top-level step after aggregation (e.g. after JQ_FILTER or PROMPT), same as GOOGLE_DRIVE_WRITE.
- **Returns:** `{item_type, id, name, web_url, drive_id, relative_path, size_bytes, c_tag, source_version}` for the created/updated SharePoint item. `id` is the Graph item id and `source_version` is the item's change token (`cTag` → `eTag` → `""`) — the same identity the inbound File Sync source computes, so a downstream sync-coherence step can register the write and the next inbound sweep skips it rather than re-ingesting the deliverable.

Typical pattern:
1. A prior step (e.g. PROMPT, JQ_FILTER, or a step that reads an approved deliverable from the docstore) produces the file content.
2. SHAREPOINT_WRITE with `input` set to that step's id, a `file_name`, and a `folder_path` for the destination folder (e.g. the client's deliverables folder).

Pipeline pattern when destination is SharePoint:
  … → SHAREPOINT_WRITE (connector_id from destinations / same SharePoint connector) → END
