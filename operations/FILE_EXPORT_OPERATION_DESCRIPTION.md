Define a FILE_EXPORT operation step in a JSON workflow language that writes structured JSON data from a previous step to a downloadable file (Excel, CSV, or PDF) and returns a presigned URL.

Basic Structure:
{
  "id": string,
  "operation": "FILE_EXPORT",
  "input": string,
  "file_format": "excel" | "csv" | "pdf" (optional, default "excel"),
  "filename": string (optional, default "export"),
  "columns": object (optional),
  "sheets": array (optional)
}

Description:
- Reads structured JSON data from intermediate_results using the 'input' key (references a previous step id)
- Builds a file in memory (Excel, CSV, or PDF), uploads to cloud storage, and returns a presigned download URL valid for 1 year
- Supports three input shapes: flat list of dicts (single sheet), dict of lists (multi-sheet), or a scalar value
- Column display names and ordering are controlled via the 'columns' map; per-sheet columns override the top-level setting
- Returns a result dict with download_url, file_name, file_type, file_size, and sheets_written

Key Features:
- Excel output: multi-tab .xlsx with bold frozen headers, auto-fit column widths, and severity-based row coloring (CRITICAL=light red, WARNING=light amber) when a "Severity" column is present
- CSV output: single .csv for one sheet; a .zip containing one .csv per sheet for multiple sheets
- PDF output: landscape A4 with dark-blue header rows, proportionally sized columns, and severity-based row coloring matching Excel; each sheet becomes a titled section on a new page with automatic page breaks and header repetition
- Column renaming: the 'columns' map defines both display order and human-readable names (keys = JSON field names, values = display names)
- skip_if_empty: per-sheet flag to omit a tab when its data array is empty
- Nested cell values (dict/list) are serialised as JSON strings

Parameters:
- input (required): Step id whose result is the data source. Accepts a list of dicts (Mode 1 — single sheet), a dict of lists (Mode 2 — one sheet per key), or a scalar.
- file_format (optional): Output format. One of "excel" (default), "csv", or "pdf".
- filename (optional): Output filename without extension (default "export"). Extension is added automatically based on file_format.
- columns (optional): Top-level column rename and ordering applied to all sheets. Object mapping JSON field names to display names. Overridden by per-sheet columns if present.
- sheets (optional): Array of sheet configuration objects for dict inputs. Each entry supports: name (display name), key (dict key to pull data from), columns (per-sheet column map), skip_if_empty (bool).

Examples:

Example 1 — Export a flat list to Excel:
{
  "id": "export_accounts",
  "operation": "FILE_EXPORT",
  "input": "fetch_accounts",
  "file_format": "excel",
  "filename": "accounts_list",
  "columns": {
    "account_name": "Account Name",
    "revenue": "Annual Revenue",
    "industry": "Industry"
  }
}

Example 2 — Multi-tab validation report with severity coloring:
{
  "id": "export_report",
  "operation": "FILE_EXPORT",
  "input": "prepare_report",
  "file_format": "excel",
  "filename": "validation_report",
  "sheets": [
    { "name": "Summary", "key": "summary" },
    { "name": "HS Code Issues", "key": "hs_code_issues", "skip_if_empty": true },
    { "name": "Missing Fields", "key": "missing_fields", "skip_if_empty": true },
    { "name": "Arithmetic Errors", "key": "arithmetic_errors", "skip_if_empty": true }
  ]
}

Example 3 — PDF report for email attachment:
{
  "id": "export_pdf",
  "operation": "FILE_EXPORT",
  "input": "prepare_report",
  "file_format": "pdf",
  "filename": "customs_validation_report"
}

Implementation Notes:
- The result dict is also stored in executor metadata["download"] so the front-end can surface it as a downloadable file in the UI
- FILE_EXPORT can appear anywhere in the workflow, not only as the last step — useful when a report is generated mid-flow for user review before proceeding
- For dict inputs without a sheets config, one tab is created per key automatically (Mode 2 auto)
- Sheet names are sanitised (forbidden Excel characters replaced with underscores) and truncated to 31 characters
- Requires unique operation ID
