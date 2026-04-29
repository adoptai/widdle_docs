Define an EMBEDDER operation step in a JSON workflow language that reads data from a source (table or S3 documents), chunks text, generates vector embeddings, and stores the vectors into a vector table for downstream similarity search.

Basic Structure:
{
  "id": string,
  "operation": "EMBEDDER",
  "input_source_type": string,
  "input_source_info": object,
  "output_table_name": string (optional),
  "config": object (optional),
  "table_label": string (required for pipeline WDLs; human-readable label for the output vector table, e.g. "documents_vec"; must be unique across all S3_READ and EMBEDDER steps in this WDL; auto-derived as {s3_read_label}_vec for S3-source steps if omitted)
}
Note: table_label is required for the output vector table (e.g. "documents_vec"). Do NOT include table_id in the WDL — it is never stored in steps. If table_label is omitted for an S3-source EMBEDDER, the system derives it automatically as {s3_read_label}_vec from the preceding S3_READ step. table_label must be unique across all S3_READ and EMBEDDER steps in the same WDL.

Key Features:
- Supports three input_source_type values: "table" (DB rows), "s3" (PDF documents from S3), and "sharepoint" (SharePoint documents)
- For "table": reads rows from a source table and converts specified columns to embeddable text
- Splits text into chunks using configurable chunk_size and chunk_overlap parameters
- Generates embeddings using configurable provider (Titan, OpenAI, etc.)
- Stores vectors in a SingleStore vector table with HNSW index for fast similarity search
- Supports full re-embed (atomic swap via temp table) and incremental mode
- Each source row's id is preserved in vector metadata for traceability
- output_table_name defaults to {resolved_input_table_name}_vec if not specified (uses the physical table name, not the label)
- output_table_name overrides the default when provided explicitly
- Input table can be referenced by table_label or table_name in input_source_info (resolution priority in that order; table_id also accepted for backward compat)
- The output vector table is automatically registered in the pipeline table registry at runtime using table_label; the system reuses an existing entry or allocates a new table_id
- For "s3": queries db_org_doc_documents by pipeline_id for PDFs with status 'queued_for_indexing', downloads from S3, parses PDF text, chunks, and embeds
- S3 source updates each document's status per-document (indexing -> indexed/error) for progress visibility
- S3 source output table name is resolved at runtime from table_label via the pipeline table registry

Examples:

1. Basic Table Embedding (by table_name):
{
  "id": "embedDocuments",
  "operation": "EMBEDDER",
  "input_source_type": "table",
  "input_source_info": {
    "table_name": "pipeline_abc123_documents",
    "content_columns": ["title", "body"]
  },
  "config": {
    "embedding_type": "Titan",
    "chunk_size": 800
  }
}

2. Table Embedding by table_id (resolved from pipeline table registry):
{
  "id": "embedByTableId",
  "operation": "EMBEDDER",
  "input_source_type": "table",
  "input_source_info": {
    "table_id": "tbl_abc123",
    "content_columns": ["title", "body"]
  },
  "config": {
    "embedding_type": "Titan",
    "chunk_size": 800
  }
}

3. Table Embedding by table_label (resolved from pipeline table registry):
{
  "id": "embedByLabel",
  "operation": "EMBEDDER",
  "input_source_type": "table",
  "input_source_info": {
    "table_label": "R&D Project Documents",
    "content_columns": ["title", "body"]
  },
  "config": {
    "embedding_type": "Titan",
    "chunk_size": 800
  }
}

4. S3 PDF Embedding (consumes output of S3_READ):
{
  "id": "embedS3Docs",
  "operation": "EMBEDDER",
  "input_source_type": "s3",
  "input_source_info": {
    "connector_id": "s3_conn_abc"
  },
  "config": {
    "embedding_type": "Titan",
    "chunk_size": 1000
  }
}
Note: pipeline_id and org_id are injected automatically from the executor context.
The EMBEDDER queries db_org_doc_documents for PDFs with status 'queued_for_indexing'
matching the pipeline_id. connector_id can be omitted if it's retrievable from the
doc store record.

5. S3 PDF Embedding with custom output table:
{
  "id": "embedS3Custom",
  "operation": "EMBEDDER",
  "input_source_type": "s3",
  "input_source_info": {},
  "output_table_name": "my_custom_pdf_vectors",
  "config": {
    "embedding_type": "Titan",
    "chunk_size": 800,
    "chunk_overlap": 100
  }
}

6. Incremental Embedding with Custom Output:
{
  "id": "embedNewRows",
  "operation": "EMBEDDER",
  "input_source_type": "table",
  "input_source_info": {
    "table_name": "pipeline_abc123_articles"
  },
  "output_table_name": "custom_articles_vec",
  "config": {
    "embedding_type": "Titan",
    "incremental": true,
    "chunk_size": 1000,
    "chunk_overlap": 150
  }
}

Implementation Notes:
- input_source_type: "table", "s3", or "sharepoint"
- For "table", input_source_info requires one of: table_id, table_label, or table_name
  - No previous step data or explicit DB read is needed; EMBEDDER reads directly from the specified table
  - table_label: resolved at runtime via registry.resolve_by_label scoped to the pipeline (preferred in pipeline WDLs)
  - table_name: used as-is (backward compatible, useful for standalone / non-pipeline use)
  - table_id: also accepted for backward compat but should not be used in new WDLs
- content_columns is optional (defaults to all columns concatenated as "key: value" pairs)
- config.embedding_type is required and must match the provider used for search
- config.incremental defaults to false for table source (atomic table swap); for S3 and SharePoint sources, incremental is always forced to true since only new documents (status 'queued_for_indexing') are embedded per run
- Pipeline tables always have an id column used as row identifier in vector metadata
- The output vector table is auto-registered in db_org_pipeline_table_registry at runtime using table_label. The physical name follows {source_table_name}_vec convention (e.g. pipeline_{pipeline_id}_{source_table_id}_vec); the registry entry maps table_label → this physical name
- In test_mode, a _test suffix is appended to the resolved table name
- Errors during embedding are caught and returned as error messages
- For "s3", input_source_info accepts: connector_id (optional if resolvable from store), region (optional)
  - pipeline_id and org_id are injected from the executor at runtime
  - Queries db_org_doc_documents WHERE pipeline_id=? AND file_type='pdf' AND status='queued_for_indexing'
  - Downloads each PDF via S3StorageClient.download_data, parses with PyPDFLoader
  - Updates db_org_doc_documents.status per-document: 'indexing' -> 'indexed' (with chunk_count, indexed_at) or 'error'
  - Vector metadata includes: source, s3_url, doc_id, filename, pipeline_id, chunk_index
  - S3_READ must run before EMBEDDER to populate db_org_doc_documents with pipeline_id
- For "sharepoint", input_source_info accepts: connector_id (reference to SharePoint connector), pipeline_id, org_id (latter two injected from executor at runtime)
  - Queries db_org_doc_documents WHERE pipeline_id=? AND file_type='pdf' AND status='queued_for_indexing'
  - Downloads each document via SharePointGraphClient, parses with PyPDFLoader
  - Updates db_org_doc_documents.status per-document: 'indexing' -> 'indexed' (with chunk_count, indexed_at) or 'error'
  - Vector metadata includes: source, sharepoint_url, doc_id, filename, pipeline_id, chunk_index
  - SHAREPOINT_READ must run before EMBEDDER to populate db_org_doc_documents with pipeline_id
