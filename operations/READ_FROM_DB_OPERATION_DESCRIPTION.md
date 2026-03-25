Defines a READ_FROM_DB step that reads data from a database using a connector.

For pipeline-created tables (same pipeline), use table_label as the user-facing short name. The system resolves table_label to a physical table at runtime via the registry; do not include table_id in the WDL step.

For cross-pipeline reads (reading a table created by a different pipeline in the same org), set table_label to the source table's full physical name (e.g. "pipeline_0508af7913dd4a8d_f293c6a7c354466a" from the sources list). The system resolves it to the existing registry entry without creating a new one.

For the internal data store (type "internal_data_store"), set connector_id to "internal_data_store" or omit it entirely; the system uses the default internal connection.

Basic Structure (step has operation "READ_FROM_DB"; all keys at the top level of the step, not nested under params):
{
  "id": string,
  "operation": "READ_FROM_DB",
  "connector_id": string (reference to DB connector; use "internal_data_store" or omit for the default internal store),
  "table_label": string (for same-pipeline tables: user-facing short name, max 64 chars; for cross-pipeline tables: the full physical table name from the sources list, e.g. "pipeline_{source_pipeline_id}_{table_id}"),
  "query": Optional(string) (SQL query; overrides table_label when provided),
  "limit": number (optional),
  "notes": string
}

Key Features:
- Reads rows from a database table or runs a query via configured connector
- Use as source step when pipeline input is in a database
- For cross-pipeline reads, set table_label to the source's table_name from the sources list; the system resolves it without a new registry entry
- connector_id "internal_data_store" (or absent) routes to the default internal data store connection
- Produces array of rows for downstream steps

Example (same-pipeline):
{ "connector_id": "internal_data_store", "table_label": "processed_results", "limit": 10000 }

Example (cross-pipeline):
{ "connector_id": "internal_data_store", "table_label": "pipeline_0508af7913dd4a8d_f293c6a7c354466a", "limit": 10000 }
