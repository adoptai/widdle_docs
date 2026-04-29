Define a SHAREPOINT_READ operation step in a JSON workflow language that lists files from a SharePoint folder via Microsoft Graph, filters by file type and timestamp, batches them, and persists batch/document records in the doc store.

Basic Structure:
{
  "id": string,
  "operation": "SHAREPOINT_READ",
  "connector_id": string (reference to SharePoint integration connector),
  "folder_path": string (optional; folder inside the drive, default root),
  "site_id": string (optional; overrides connector-configured site),
  "drive_id": string (optional; overrides connector-configured drive),
  "site_url": string (optional; used to resolve site_id if site_id is absent),
  "store_id": string (optional; if omitted, auto-created or reused),
  "store_name": string (optional; human-readable name for auto-created store),
  "file_types": array (optional; e.g. ["pdf", "txt"], defaults to supported types),
  "since": string (optional; ISO-8601 timestamp; only files modified after this),
  "batch_limit_files": number (optional; max files per batch, default 100),
  "batch_limit_size_bytes": number (optional; max batch size in bytes, default 500MB),
  "table_label": string (optional; human-readable label for pipeline table registry)
}
Note: org_id is injected automatically from the executor context at runtime. Do NOT include org_id in the step.

Key Features:
- Uses Microsoft Graph with app credentials to read SharePoint files
- Recursively traverses all subfolders under the specified `folder_path` (or drive root if omitted) via BFS using the Graph API `/items/{id}/children` endpoint
- Filters by extension and incremental timestamp
- Batches files by count and total size
- Persists store, batch, and document metadata rows in SingleStore docstore tables
- Returns file metadata plus `adopt_internal_source_id` for downstream lineage
- Pipeline table registry: on success, creates a physical `pipeline_{pipeline_id}_{table_id}` REFERENCE TABLE in SingleStore, copies document rows into it, and marks the registry entry. Supports both `table_label` (label-only WDLs) and legacy `table_id`.
- `sync_sharepoint_pipeline_doc_table`: utility function to re-sync document statuses from `db_org_doc_documents` into the pipeline table after EMBEDDER runs (e.g. queued_for_indexing → indexed).

Examples:

1. Basic SharePoint Read:
{
  "id": "readSharepointDocs",
  "operation": "SHAREPOINT_READ",
  "connector_id": "sp_connector_123",
  "folder_path": "Shared Documents/Finance",
  "file_types": ["pdf", "txt"]
}

2. Explicit Site/Drive Override:
{
  "id": "readSharepointDrive",
  "operation": "SHAREPOINT_READ",
  "connector_id": "sp_connector_123",
  "site_id": "contoso.sharepoint.com,abc,def",
  "drive_id": "b!xyz123",
  "folder_path": "General/Reports",
  "batch_limit_files": 50
}

3. Pipeline with Table Registry (label-based):
{
  "id": "readSharepointDocs",
  "operation": "SHAREPOINT_READ",
  "connector_id": "sp_connector_123",
  "folder_path": "Shared Documents/HR",
  "table_label": "sharepoint_hr_docs"
}

Implementation Notes:
- `connector_id` is required
- `site_id`/`drive_id` can come from connector secrets or be overridden in the step
- `folder_path` is optional; drive root ("Shared Documents" in SharePoint UI) is used when omitted — all subfolders are traversed recursively regardless
- currently supported file types are `pdf` and `txt`
- each output row includes `adopt_internal_source_id` (MD5 of canonical source URL)
- `table_label` is preferred over legacy `table_id` for pipeline table registry; if neither is present, registry creation is skipped silently
