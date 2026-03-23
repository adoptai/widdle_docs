Defines a FAN_OUT step for per-row processing: take an array of rows from a previous step, run a sequence of WDL steps on each row individually, then gather the results back into an array (scatter–gather loop).

Use when you need to process each row with its own chain of operations (e.g. JQ_FILTER, PROMPT, TOOL_EXECUTION per row) and then continue the pipeline with the combined array.

Basic Structure (step has operation "FAN_OUT"; key-value pairs below):
{
  "id": string,
  "operation": "FAN_OUT",
  "input": string (id of previous step that produces an array of rows),
  "sub_steps": array of step objects (WDL steps to run per row; each step receives one row as input; outputs are gathered in order),
  "batch_size": number (optional; default 25; when write_step is set, reads from DB in batches of this size—avoids loading full array into memory),
  "write_step": string (optional; step id of WRITE_TO_DB step; required for batch_size mode),
  "notes": string
}

Key Features:
- input: step id whose output is an array of rows (e.g. from READ_FROM_DB or a prior JQ_FILTER).
- sub_steps: list of pipeline steps. When running via PipelineWorkflowExecutor (pipeline test/live runs), only pipeline-allowed operations may be used (see ALLOWED_PIPELINE_STEP_TYPES). When running via WiddleExecutor (actionbot), any WDL operation may be used. Each sub-step runs once per row; the row is the input context for that iteration. Order of sub_steps is preserved; the first sub-step receives the row, later sub-steps receive the output of the previous sub-step for that row.
- fan_out_row: the unmodified original row for the current iteration is always available as "fan_out_row" in intermediate_results throughout all sub_steps. Use this when a sub-step transforms the data but a later sub-step still needs the original row fields (e.g. reference the source id or key from the original row after a PROMPT or JQ_FILTER has changed the shape).
- Gather is implicit: after all rows are processed, the step output is the array of results (one per row, in original order).
- Use for: per-row enrichment, per-row LLM calls, per-row transforms, then aggregating back into a single array for downstream steps.

Example (first sub_step "input" must be the FAN_OUT input step id so it receives the current row):
{ "id": "fanout_enrich", "operation": "FAN_OUT", "input": "step_read", "sub_steps": [
  { "id": "loop_jq", "operation": "JQ_FILTER", "input": "step_read", "filter": ".", "extract_all": false },
  { "id": "loop_prompt", "operation": "PROMPT", "input": "loop_jq", "prompt_template": "Summarize: {{.}}" }
], "notes": "Enrich each row with a summary then gather" }

---

## Leveraging FAN_OUT in pipelines with other operations

**Upstream (feed array into FAN_OUT):**
- **READ_FROM_DB** — Produces an array of rows; use its step id as FAN_OUT `input`.
- **JQ_FILTER** — If the step outputs an array (e.g. `extract_all: true` or filter that returns a list), use its step id as FAN_OUT `input`.
- Any step that writes an array into `intermediate_results` can be the FAN_OUT `input`.

**Inside FAN_OUT (sub_steps, per row):**
- **JQ_FILTER** — Transform or extract fields from the row; first sub_step `input` = FAN_OUT `input` step id (the row).
- **PROMPT** — LLM call per row (e.g. summarize, classify, translate).
- **TOOL_EXECUTION** — Call an integration tool per row.
- **CONDITION**, **EDIT_VALUE**, **CONDITIONAL_ASSIGNMENT**, **FILTER**, **EXTRACT**, **MERGE**, etc. — Any WDL operation in SINGLE_OP_DISPATCH can be used; sub_steps run in order, each sees the previous sub_step's result via its `input` step id.
- To reference the original unmodified row at any point within sub_steps, use `"fan_out_row"` as the `input` step id (e.g. `"input": "fan_out_row"` in a JQ_FILTER or MERGE sub-step).

**CRITICAL — Merging PROMPT output with row metadata:**
A PROMPT step with `fields: ["summary"]` returns ONLY `{"summary": "..."}` — it does NOT carry forward any other fields from its input. If a downstream JQ_FILTER needs both the PROMPT-generated value AND original row fields (e.g. mail_id, subject), use `inputs` (plural) on the JQ_FILTER to reference both sub_steps simultaneously. The executor combines them into `{<stepA_id>: <stepA_output>, <stepB_id>: <stepB_output>}` so the filter can access both.

Example — summarize each email and preserve its metadata:
```json
{ "id": "fanout_summarize", "operation": "FAN_OUT", "input": "extract_emails", "sub_steps": [
  { "id": "extract_email_data", "operation": "JQ_FILTER", "input": "extract_emails",
    "filter": "{mail_id: .mail_id, subject: .subject, mail_content: .mail_content}",
    "extract_all": false, "notes": "Extract fields for PROMPT" },
  { "id": "summarize_email", "operation": "PROMPT", "input": "extract_email_data",
    "instructions": "Summarize the email in one sentence.", "is_last_step": false,
    "fields": ["summary"], "notes": "Generate one-sentence summary" },
  { "id": "combine_with_metadata", "operation": "JQ_FILTER",
    "inputs": ["extract_email_data", "summarize_email"],
    "filter": "{mail_id: .extract_email_data.mail_id, subject: .extract_email_data.subject, summary: .summarize_email.summary}",
    "extract_all": false, "notes": "Merge summary with original metadata" }
], "notes": "Summarize each email and attach metadata" }
```
All prior sub_step results are available by their step id in `inputs`; any earlier sub_step id can be referenced.

**Downstream (use FAN_OUT output):**
- **WRITE_TO_DB** — Use FAN_OUT step id as `input`; writes the gathered array (one element per row).
- **JQ_FILTER** — Further transform or filter the gathered array.
- **CONDITION** — Branch on the gathered array.
- Another **FAN_OUT** — Nested fan-out (FAN_OUT `input` = previous FAN_OUT step id).

**Typical pipeline flow:**
1. READ_FROM_DB (or source) → array of rows
2. Optional: JQ_FILTER / CONDITION on the full array
3. FAN_OUT (input = step that has the array) with sub_steps e.g. JQ_FILTER → PROMPT → TOOL_EXECUTION per row
4. Optional: JQ_FILTER / CONDITION on the gathered array
5. WRITE_TO_DB (input = FAN_OUT step id or last transform step)

---

## Batch mode: memory-efficient read → process → write

**Use case:** When READ_FROM_DB feeds a large array into FAN_OUT and the result is written to DB, the default flow loads the entire array into memory, causing high memory usage and temporal issues.

**Solution:** Set `batch_size` and `write_step` on FAN_OUT. The executor will:
1. Skip the READ_FROM_DB step (no full load)
2. Read from DB in batches of `batch_size`
3. Process each batch with sub_steps
4. Write each batch to DB immediately
5. Never hold more than `batch_size` rows in memory

**Requirements:**
- FAN_OUT `input` must reference the READ_FROM_DB step id
- READ_FROM_DB must be the immediate previous step (no JQ_FILTER or other step in between)
- `write_step` must be the id of the WRITE_TO_DB step (which has `input` = FAN_OUT step id)

**Example WDL:**
```json
[
  {"id": "step_read", "operation": "READ_FROM_DB", "table_id": "source", "connector_id": "internal_data_store"},
  {"id": "fanout", "operation": "FAN_OUT", "input": "step_read", "write_step": "step_write", "sub_steps": [
    {"id": "jq", "operation": "JQ_FILTER", "input": "step_read", "filter": "{summary: .data.summary}", "extract_all": false},
    {"id": "prompt", "operation": "PROMPT", "input": "jq", "instructions": "Summarize", "fields": ["summary"]}
  ]},
  {"id": "step_write", "operation": "WRITE_TO_DB", "input": "fanout", "table_id": "dest"}
]
```

**Output:** In batch mode, FAN_OUT returns `[]` (no gathered array for downstream). All data is written incrementally to the destination table.
