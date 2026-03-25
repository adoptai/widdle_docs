Define a DOCUMENT_EXTRACTOR operation step in a JSON workflow language that extracts structured data from uploaded files (PDF, Excel, Word, images) using an LLM gateway.

Basic Structure:
{
  "id": string,
  "operation": "DOCUMENT_EXTRACTOR",
  "input": string,
  "fields_to_extract": string[] (optional - at least one of fields_to_extract or field_schemas required),
  "field_schemas": object (optional - at least one of fields_to_extract or field_schemas required),
  "instructions": string (optional),
  "required_doc_types": object (optional),
  "conditional_doc_types": object (optional)
}

Description:
- Downloads file(s) referenced by the 'input' key from file_uploads (provided via an ASK_USER_FOR_INPUT step with FILE_UPLOAD)
- Routes each file through an LLM gateway for structured field extraction
- Supports PDF (text extraction via pypdf), Excel XLSX (multi-sheet CSV conversion via openpyxl), Excel XLS (via xlrd), Word DOCX (paragraphs and tables via python-docx), and images (base64-encoded, sent as image_url)
- Returns a single dict when one file is uploaded (backward compatible), or a list of dicts when multiple files are uploaded (batch mode)
- Each batch result is tagged with `_source_file` (original filename) and `_file_index` (position in upload order)
- Includes an optional filename-based pre-check to verify all required document types are present before any LLM calls

Key Features:
- LLM-powered structured extraction from trade documents (invoices, bills of lading, packing lists, order forms, etc.)
- Supports flat fields (strings, numbers, dates) and table-like schemas (arrays of objects with named columns)
- Batch processing: when input references a list of files, each file is processed sequentially and results are returned as a list
- Filename pre-check via required_doc_types: verifies all expected document types are present by matching keywords against filenames, avoiding wasted LLM calls on incomplete uploads
- Dynamic document requirements via conditional_doc_types: adds extra required types based on a condition from a previous workflow step (e.g., requiring an LOA document only when no existing relationship exists)
- File size guard: rejects files larger than 20 MB
- Empty content guard: skips LLM call if extracted text/CSV content is empty
- Per-call timeout: each LLM call has a 120-second timeout
- Requires unique operation ID

Parameters:
- input (required): References an ASK_USER_FOR_INPUT step field id whose FILE_UPLOAD field contains the uploaded file(s). Can reference a single file (dict) or multiple files (list of dicts).
- fields_to_extract (optional): List of field names to extract from the document (e.g., ["importer_name", "invoice_number", "country_of_origin"]). At least one of fields_to_extract or field_schemas must be provided.
- field_schemas (optional): Object mapping field names to schema definitions for structured/tabular data. Each entry has a "type" (e.g., "table") and "columns" (list of column names). At least one of fields_to_extract or field_schemas must be provided.
- instructions (optional): Additional extraction instructions appended to the LLM prompt (e.g., "Amounts are in USD. Use ISO date format YYYY-MM-DD.").
- required_doc_types (optional): Object mapping document type names to lists of filename keywords. Before any LLM calls, filenames are checked against these keywords. If any type has no matching file, the operation returns an error listing the missing types.
- conditional_doc_types (optional): Object with "condition" (a dot-path reference like "step_id.field") and "doc_types" (same format as required_doc_types). If the referenced field evaluates to true/yes/1, the extra doc_types are merged into required_doc_types.

Examples:

1. Single File Extraction (Invoice):
{
  "id": "parse_invoice",
  "operation": "DOCUMENT_EXTRACTOR",
  "input": "trade_docs",
  "fields_to_extract": ["importer_name", "exporter_name", "invoice_number", "invoice_date", "country_of_origin", "total_amount", "currency"],
  "field_schemas": {
    "line_items": {
      "type": "table",
      "columns": ["hs_code", "description", "quantity", "unit_price", "total_price"]
    }
  },
  "instructions": "Amounts are in USD. Use ISO date format YYYY-MM-DD."
}

2. Batch Extraction with Document Type Pre-Check:
{
  "id": "parse_docs",
  "operation": "DOCUMENT_EXTRACTOR",
  "input": "trade_docs",
  "fields_to_extract": ["document_type", "importer_name", "exporter_name", "invoice_number", "bl_number", "vessel_name", "port_of_loading", "gross_weight"],
  "field_schemas": {
    "line_items": {
      "type": "table",
      "columns": ["hs_code", "description", "quantity", "unit_price", "total_price"]
    }
  },
  "required_doc_types": {
    "invoice": ["invoice", "commercial invoice"],
    "packing_list": ["packing list", "packing_list"],
    "bill_of_lading": ["bol", "bill of lading", "mbl"],
    "order_form": ["order form", "order_form"]
  }
}

3. Conditional Document Requirement (LOA):
{
  "id": "parse_docs",
  "operation": "DOCUMENT_EXTRACTOR",
  "input": "trade_docs",
  "fields_to_extract": ["document_type", "importer_name", "exporter_name"],
  "required_doc_types": {
    "invoice": ["invoice"],
    "packing_list": ["packing list"],
    "bill_of_lading": ["bol", "bill of lading"],
    "order_form": ["order form"]
  },
  "conditional_doc_types": {
    "condition": "check_relationship.loa_required",
    "doc_types": {
      "loa": ["loa", "letter of authorization", "letter of authority"]
    }
  }
}

Output Shape:
- Single file: a dict with extracted field names as keys and extracted values as values. Table fields (from field_schemas) are arrays of objects.
- Batch (multiple files): a list of dicts, each tagged with "_source_file" and "_file_index". Results are in the original upload order.
- On error: (None, error_message_string)

Implementation Notes:
- Requires a valid config_instance with LLM gateway credentials (llm_gateway_api_key, llm_gateway_url)
- The 'input' key must reference an ASK_USER_FOR_INPUT step that has a FILE_UPLOAD field
- Supported file types: .pdf, .xlsx, .xls, .docx, .png, .jpg, .jpeg, .gif, .bmp, .tiff, .tif, .webp
- For Excel files (.xlsx via openpyxl, .xls via xlrd), ALL sheets are converted to CSV and sent as a single text block to the LLM
- For Word documents (.docx), paragraphs and tables are extracted as text
- For images, file bytes are base64-encoded and sent as an image_url message
- If a file in a batch fails (download error, unsupported type, LLM error), it is skipped and other files continue processing
- If ALL files in a batch fail, the operation returns an error
- The operation stores its result in intermediate_results under the step id, making it available to subsequent steps via {step_id} references
