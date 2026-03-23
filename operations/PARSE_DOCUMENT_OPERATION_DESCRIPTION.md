Define a PARSE_DOCUMENT operation step in a JSON workflow language that downloads and parses file(s) from S3 for use by downstream PROMPT or AGENTIC_SEARCH steps.

Basic Structure:
{
  "id": string,
  "operation": "PARSE_DOCUMENT",
  "connector_id": string (reference to S3 pipeline connector — same as the pipeline's source connector_id),
  "file_path": string (see File Path Rules below)
}

File Path Rules:
- file_path is RELATIVE to the connector's S3 prefix (the path component of BucketURI). Do NOT repeat the bucket name or the connector prefix in file_path — it is automatically prepended.
  Example: if BucketURI = "s3://my-bucket/org_data/docs/" and the file is at s3://my-bucket/org_data/docs/report.pdf, then file_path = "report.pdf" (NOT "org_data/docs/report.pdf").
- When the user does NOT specify a particular file name, use file_path = "*" to process ALL supported files in the connector's S3 path. This is the most common pattern for pipelines wired to an S3 source.
- Glob patterns are supported: "*.pdf" (all PDFs), "reports/*.csv" (CSVs in reports/ subfolder).
- Template variables are supported: "{{previousStep.relative_path}}" resolves at runtime.

Output (single file — file_path is a concrete path):
{
  "text": string,
  "pages": [{"page_number": int, "text": string}, ...],
  "filename": string,
  "file_type": "pdf" | "csv" | "txt" | "json" | "md" | "xml" | "html",
  "file_path": string,
  "s3_url": string,
  "page_count": int
}

Output (wildcard / glob — file_path contains * or ?):
{
  "documents": [
    {"text": string, "pages": [...], "filename": string, "file_type": string, "file_path": string, "s3_url": string, "page_count": int},
    ...
  ],
  "total_files": int,
  "failed_files": int,
  "file_path": string (the original pattern)
}

Key Features:
- Downloads files from S3 using pipeline connector credentials
- Supports PDF (extracts pages with page numbers), text, CSV, JSON, XML, HTML
- PDF parsing uses PyPDFLoader for reliable page-level text extraction
- connector_id can be specified in the step or inherited from workflow_arguments
- Wildcard mode processes all matching files and returns a documents array

Examples:

1. Parse ALL files in the connector's S3 path (most common for S3-source pipelines):
{
  "id": "parseAllDocs",
  "operation": "PARSE_DOCUMENT",
  "connector_id": "s3-connector-01",
  "file_path": "*"
}

2. Parse only PDF files:
{
  "id": "parsePdfs",
  "operation": "PARSE_DOCUMENT",
  "connector_id": "s3-connector-01",
  "file_path": "*.pdf"
}

3. Parse a specific file:
{
  "id": "parseDoc",
  "operation": "PARSE_DOCUMENT",
  "connector_id": "s3-connector-01",
  "file_path": "sample_tax_form.pdf"
}

4. Parse with Template Variable from Previous Step:
{
  "id": "parseDynamic",
  "operation": "PARSE_DOCUMENT",
  "connector_id": "s3-connector-01",
  "file_path": "{{s3Read.files[0].relative_path}}"
}

Implementation Notes:
- connector_id and file_path are required; org_id is injected from the executor context
- For PDF files, the output includes a "pages" array with page_number and text for each page
- For non-PDF files, the "pages" array is empty and page_count is 1
- The full S3 key is: {connector_prefix}/{file_path} where connector_prefix comes from the BucketURI
- Credentials are resolved from db_org_pipeline_connector via the connector_id
- If text extraction fails or returns empty, an error is returned
- When using wildcard, partially failed files are skipped (errors logged); the operation succeeds if at least one file parses
