Define an S3_READ operation step in a JSON workflow language that lists objects from an S3 folder, filters by file type and timestamp, batches them, and persists batch and document records in the doc store.

Basic Structure:
{
  "id": string,
  "operation": "S3_READ",
  "connector_id": string (reference to S3 integration connector),
  "s3_folder_path": string (S3 prefix or full URI, e.g. "org_id=abc/" or "s3://bucket/prefix/"),
  "store_id": string (optional; if omitted, auto-created or reused based on connector + org + bucket + prefix),
  "store_name": string (optional; human-readable name for auto-created store),
  "file_types": array (optional; e.g. ["pdf", "txt"], defaults to ["pdf", "txt"]),
  "region": string (optional; AWS region, defaults to "us-east-1"),
  "batch_limit_files": number (optional; max files per batch, default 100),
  "batch_limit_size_bytes": number (optional; max batch size in bytes, default 500MB),
  "table_label": string (optional; human-readable label for pipeline table registry)
}
Note: Always include table_label (a short human-readable name, e.g. "s3_documents") so the system can resolve or create the physical table at runtime. Do NOT include table_id in the WDL — it is never stored in steps and is resolved entirely at runtime.
Note: org_id is injected automatically from the executor context at runtime. Do NOT include org_id in the step.

Key Features:
- Lists objects recursively from the given S3 prefix
- Supports full S3 URIs (s3://bucket/path) to override the bucket from connector credentials
- Filters files by extension (file_types) and by last_sync_timestamp for incremental syncs
- Batches files by count and size limits
- Persists doc store, batch, and document metadata records in SingleStore
- Auth credentials are resolved at runtime using connector_id and the executor-provided org_id
- If store_id is not provided, finds or creates a store based on (connector_id, org_id, s3_bucket, s3_prefix)
- Returns list of file paths, batch info, store_id, and sync statistics

Examples:

1. Basic S3 Read with Auto Store:
{
  "id": "readS3Documents",
  "operation": "S3_READ",
  "connector_id": "s3_connector_456",
  "s3_folder_path": "org_id=org_123/documents/",
  "file_types": ["pdf", "txt"]
}

2. Full URI with Custom Batch Limits:
{
  "id": "readLargeDataset",
  "operation": "S3_READ",
  "connector_id": "s3_connector_456",
  "s3_folder_path": "s3://my-bucket/data/quarterly/",
  "batch_limit_files": 50,
  "batch_limit_size_bytes": 268435456,
  "region": "us-west-2"
}

3. With Existing Store:
{
  "id": "incrementalSync",
  "operation": "S3_READ",
  "connector_id": "s3_connector_456",
  "s3_folder_path": "reports/",
  "store_id": "existing_store_789",
  "file_types": ["pdf"]
}

Implementation Notes:
- table_label is required and must be unique across all S3_READ and EMBEDDER steps in the same WDL. At runtime the system resolves table_label → existing registry entry (reuse) or allocates a new table_id and creates pipeline_{pipeline_id}_{table_id}. Actual document data lives in the docstore tables.
- connector_id and s3_folder_path are required; org_id is injected from the executor
- s3_folder_path supports both relative prefix and full s3:// URI
- file_types defaults to ["pdf", "txt"]; only these extensions are currently supported
- Incremental sync uses last_sync_timestamp from the store to skip previously synced files
- If the same file appears again (same key), it is added as a new document record
- Batch records track file count, total size, and processing status
- Errors during S3 listing or credential resolution are returned as error messages
