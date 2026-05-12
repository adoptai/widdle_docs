Defines a WRITE_TO_DB step that writes pipeline output to a database.

---

## Choosing the right destination

| Signal from user / workstream | Use |
|---|---|
| General pipeline output, no Data Store context | Destination 1 (default, SingleStore) |
| Workstream explicitly tied to a structured data store (Tax, AP, invoices, expenses, etc.) | Destination 2 (`db: "org_internal"`) |
| User says "write to my database / data store / org DB" | Destination 2 (`db: "org_internal"`) |
| User has a pre-defined table in their Data Store | Destination 2 with `table_name` (typed write) |
| No pre-defined table; pipeline should own and create its output table | Destination 2 without `table_name` (auto write) OR Destination 1 |

---

## Destination 1: Internal data store (SingleStore / MySQL) — default

Omit `db`, or set `connector_id: "internal_data_store"`. The system auto-resolves or creates a physical table from `table_label`.

Fields:
```
id            string   — step id
operation     "WRITE_TO_DB"
input         string   — id of the upstream step whose output to write
connector_id  string   — use "internal_data_store" or omit
table_label   string   — user-facing short name (required; unique per WDL; max 64 chars)
table_purpose string   — human-readable description (max 64 chars)
mode          string   — optional; "append" (default) | "replace" | "upsert"
notes         string   — optional
```

Rules:
- `table_label` is required and must be unique across all WRITE_TO_DB steps in the same WDL.
- Do NOT include `table_id` in the WDL — the system resolves or allocates it at runtime.
- Physical table name is `pipeline_{pipeline_id}_{table_id}` (opaque to users and AI).

Example — full pipeline shape (read → transform → write):
```json
[
  {
    "id": "read_emails",
    "operation": "OUTLOOK",
    "folder": "Inbox",
    "limit": 100
  },
  {
    "id": "summarize",
    "operation": "PROMPT",
    "input": "read_emails",
    "prompt": "Summarize the email. Keep adopt_internal_source_id unchanged.",
    "output_fields": ["adopt_internal_source_id", "subject", "summary"]
  },
  {
    "id": "write_results",
    "operation": "WRITE_TO_DB",
    "input": "summarize",
    "connector_id": "internal_data_store",
    "table_label": "email_summaries",
    "table_purpose": "Inbox emails with AI-generated summaries",
    "mode": "upsert"
  }
]
```

---

## Destination 2: Org Data Store (Postgres) — `"db": "org_internal"`

Set `"db": "org_internal"` to write to the org's dedicated Postgres database (Data Store module). Two sub-modes:

### A. Typed write — `table_name` provided

Use when the org already has a user-defined table in their Data Store (created via the UI). The table's `column_schema` governs which fields are written. Extra fields not in the schema are merged into a `metadata` JSONB column if one exists, otherwise silently dropped. Hard failure if the table does not exist or is not active.

Fields:
```
id          string   — step id
operation   "WRITE_TO_DB"
input       string   — id of the upstream step
db          "org_internal"
table_name  string   — exact table name (must match ^[a-z][a-z0-9_]{0,62}$; must exist in Data Store)
mode        string   — optional; "append" (default) | "replace" | "upsert"
notes       string   — optional
```

Example — full pipeline shape:
```json
[
  {
    "id": "fetch_invoices",
    "operation": "READ_FROM_DB",
    "connector_id": "erp_connector",
    "query": "SELECT id AS adopt_internal_source_id, vendor, amount, due_date FROM invoices WHERE status = 'pending'"
  },
  {
    "id": "enrich",
    "operation": "PROMPT",
    "input": "fetch_invoices",
    "prompt": "Classify risk level for this invoice. Keep adopt_internal_source_id unchanged.",
    "output_fields": ["adopt_internal_source_id", "vendor", "amount", "due_date", "risk_level"]
  },
  {
    "id": "write_invoices",
    "operation": "WRITE_TO_DB",
    "input": "enrich",
    "db": "org_internal",
    "table_name": "invoices",
    "mode": "upsert"
  }
]
```

### B. Standard write — no `table_name`

Use when the pipeline should own its output table. The system auto-creates `pipeline_{sanitized_pipeline_id}` (lowercased, non-alphanumeric replaced with `_`, truncated to 63 chars total) on first write using the 10-column standard schema (id, pipeline_id, workstream_id, source_id, data JSONB, created_at, created_by, updated_at, updated_by, is_deleted). The table and a writer binding are auto-registered in the Data Store so it appears in the UI.

Fields:
```
id        string   — step id
operation "WRITE_TO_DB"
input     string   — id of the upstream step
db        "org_internal"
mode      string   — optional; "append" (default) | "replace" | "upsert"
notes     string   — optional
```

Example — full pipeline shape:
```json
[
  {
    "id": "read_transactions",
    "operation": "READ_FROM_DB",
    "connector_id": "accounting_connector",
    "query": "SELECT txn_id AS adopt_internal_source_id, amount, category, date FROM transactions"
  },
  {
    "id": "classify",
    "operation": "PROMPT",
    "input": "read_transactions",
    "prompt": "Classify this transaction as business or personal. Keep adopt_internal_source_id unchanged.",
    "output_fields": ["adopt_internal_source_id", "amount", "category", "date", "classification"]
  },
  {
    "id": "write_output",
    "operation": "WRITE_TO_DB",
    "input": "classify",
    "db": "org_internal",
    "mode": "append"
  }
]
```

### C. S3 / file source → OrgDB (JSON or CSV file on S3)

**Critical**: `PARSE_DOCUMENT` always returns `{"text": "<raw file content string>", "file_type": "json"|"csv", ...}` — it does NOT parse the file into an array. You MUST add a `JQ_FILTER` step using `.text | fromjson` (for JSON) or custom CSV parsing to produce the array before `WRITE_TO_DB`.

**Never use `filter: "."` after `PARSE_DOCUMENT`** — that just passes the whole `{text, filename, ...}` wrapper object through, which is not an array and will break the write step.

Example — S3 JSON file → typed OrgDB table (complete working shape):
```json
[
  {
    "id": "fetch_file",
    "operation": "PARSE_DOCUMENT",
    "connector_id": "<s3_connector_id>",
    "source_type": "s3",
    "file_path": "employees.json"
  },
  {
    "id": "parse_rows",
    "operation": "JQ_FILTER",
    "input": "fetch_file",
    "filter": ".text | fromjson | map({adopt_internal_source_id: (.employee_id | tostring)} + .)",
    "extract_all": false
  },
  {
    "id": "write_employees",
    "operation": "WRITE_TO_DB",
    "input": "parse_rows",
    "db": "org_internal",
    "table_name": "employees",
    "mode": "upsert"
  }
]
```

Rules for this pattern:
- Use a **single** `JQ_FILTER` step that both parses (`.text | fromjson`) and transforms (maps fields, sets `adopt_internal_source_id`) in one filter expression.
- Do NOT use separate steps for "extract array" and "map fields" — combine them.
- `adopt_internal_source_id` must be set from the row's natural primary key (e.g. `employee_id`, `invoice_id`) to enable upsert deduplication.
- `extract_all: false` is required because `.text | fromjson | map(...)` emits ONE result (the whole array). Using `extract_all: true` would double-wrap it into `[[...]]`.

### Key rules for `db: "org_internal"`

- The org must have an active database provisioned in the Data Store module.
- `table_name`, if given, must be a lowercase identifier matching `^[a-z][a-z0-9_]{0,62}$`.
- If `table_name` is given and the table does not exist, the pipeline fails hard — do not silently fall back.
- Do NOT set both `table_name` and `table_label` in the same step; they are mutually exclusive across destinations.

---

## Field retention (applies to both destinations)

- Every source read (READ_FROM_DB, OUTLOOK, S3_READ, etc.) returns rows with an `adopt_internal_source_id` field. This maps to the `source_id` column for lineage tracking and deduplication (`ON CONFLICT` / `ON DUPLICATE KEY UPDATE`).
- Every intermediate step (JQ_FILTER, PROMPT, FAN_OUT sub_steps, etc.) MUST carry `adopt_internal_source_id` through unchanged — never drop or rename it. All content transformations write results into other fields only.
- All DB schema fields (id, pipeline_id, source_id, created_at, created_by, updated_at, updated_by, is_deleted) are always set to pipeline defaults on write — they do not need to be carried through intermediate steps.
- DB schema fields are never embedded inside the data JSON payload.
