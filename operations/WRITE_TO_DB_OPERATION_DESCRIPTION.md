Defines a WRITE_TO_DB step that writes data to a database using a connector.

For pipeline-created tables, use table_label (user-facing short name) and table_purpose (description). table_label is the sole identifier used to resolve or create the physical table at runtime — the system looks up the registry by label and allocates a new table_id if none exists. Do NOT include table_id in the WDL step. The physical table name is pipeline_{pipeline_id}_{table_id} and is opaque to users.

For the internal data store (destination type "internal_data_store"), set connector_id to "internal_data_store" or omit it entirely; the system uses the default internal connection.

Basic Structure (step has operation "WRITE_TO_DB"; all keys at the top level of the step, not nested under params):
{
  "id": string,
  "operation": "WRITE_TO_DB",
  "input": string (id of previous step),
  "connector_id": string (reference to DB connector; use "internal_data_store" or omit for the default internal store),
  "table_label": string (user-facing short name for this output, e.g. "gmail_messages_with_summaries"; max 64 chars),
  "table_purpose": string (human-readable description of what this output contains, stored in registry; max 64 chars),
  "mode": string (optional; default "append"; allowed values: "append", "replace", "upsert"),
  "notes": string
}

Key Features:
- Writes output of a previous step to a database table
- Use as destination step when pipeline output should be persisted to a database
- table_label is the short user-facing name shown in the UI (e.g. "gmail_messages_with_summaries"); it is required and must be unique across all WRITE_TO_DB steps in the same WDL
- table_purpose describes what the table contains (e.g. "Gmail messages with AI-generated summaries")
- At runtime, the system resolves table_label → existing registry entry (reuse table_id) or allocates a new table_id; no table_id is stored in the WDL
- connector_id "internal_data_store" (or absent) routes to the default internal data store connection

Field retention (source → transform → WRITE flows):
- Every source read (READ_FROM_DB, OUTLOOK, S3_READ) returns rows with an "adopt_internal_source_id" field. WRITE_TO_DB maps it to the "source_id" column for lineage tracking and deduplication (ON DUPLICATE KEY UPDATE).
- Every intermediate step (JQ_FILTER, PROMPT, FAN_OUT sub_steps, etc.) MUST carry "adopt_internal_source_id" through unchanged — never drop it. All content transformations write results into the "data" field only.
- All DB schema fields (id, pipeline_id, source_id, created_at, created_by, updated_at, updated_by, is_deleted) are always reset to pipeline defaults on write — they do not need to be carried through intermediate steps.
- DB schema fields are never embedded inside the data JSON payload.

Example:
```json
{
  "id": "write_results",
  "operation": "WRITE_TO_DB",
  "input": "step_1",
  "connector_id": "internal_data_store",
  "table_label": "processed_results",
  "table_purpose": "Transformed and summarized results",
  "mode": "append"
}
```
