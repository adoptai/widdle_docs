Defines a RUN_ACTION step that executes a published action (ActionV2) as a sub-workflow within the pipeline.

Use this step when the pipeline needs to invoke a pre-built, reusable action by its action_id. The action runs in an isolated sub-executor; its output is returned as this step's result and can be used by downstream steps.

The action_id MUST come from the "Available actions" list provided in the prompt context. Do not invent or hallucinate action IDs.

All keys are at the top level of the step (not nested under "params").

Basic Structure:
{
  "id": string,
  "operation": "RUN_ACTION",
  "action_id": string (ID of the published action to run; must be from the available actions list),
  "input": object (optional; maps the action's required_inputs to values from upstream step results using {step_id.field} syntax — do NOT put user_query here),
  "user_query": string (optional; a literal query string to pass to the action — use when the pipeline prompt contains a specific search/filter query for the action to operate on; omit if the action inherits the pipeline's user query),
  "output_step_id": string (optional; see "Result type and output_step_id" below),
  "notes": string
}

Result type and output_step_id
--------------------------------
The result of a RUN_ACTION step is NOT always a list of dicts. The type depends on
what the inner action produces. The resolution order is:

1. output_step_id (explicit) — if set, the raw intermediate result of that specific
   inner step is returned unchanged. Use this when a downstream step (e.g. WRITE_TO_DB)
   needs a list of dicts and you want to target an EXTRACT or SORT step inside the action.
   IMPORTANT: the value must exactly match the "id" field of a step inside the inner
   action's WDL. This id is NOT known at pipeline-generation time and cannot be
   verified automatically — only set output_step_id when you are certain of the inner
   step id from the action definition.

2. formatted_message.data (default, structured) — when output_step_id is omitted,
   the executor checks formatted_message.data from the sub-execution state. If it is
   a list of dicts (e.g. produced by an OUTPUT_TABLE step), that list is returned.
   This is the common case and is directly usable by downstream WRITE_TO_DB steps
   without requiring an explicit output_step_id.

3. execution_output (default, text) — if formatted_message.data is not a list of
   dicts, the list of formatted output strings from all OUTPUT-type steps is returned
   (list[str]). This is NOT suitable as-is for WRITE_TO_DB.

4. formatted_message object — returned as-is if execution_output is also empty.

5. None — when the sub-executor produced no output at all.

Guidance for downstream WRITE_TO_DB:
- If the inner action has an OUTPUT_TABLE step, omit output_step_id — the structured
  rows (list[dict]) are returned automatically via formatted_message.data.
- If the inner action has no OUTPUT_TABLE step (e.g. ends with EXTRACT, or JQ_FILTER),
  set output_step_id to the id of that step (EXTRACT, or JQ_FILTER) to get the raw list[dict].
  Only do this when you know the exact inner step id.
- Alternatively, add a JQ_FILTER step in the outer pipeline immediately after RUN_ACTION
  to reshape or filter the result into list[dict] before WRITE_TO_DB. This avoids needing
  to know any inner step id and is the preferred approach when output_step_id is uncertain.

Key Features:
- Executes any published action by reference; the latest published version is always used
- input maps required_inputs to upstream step result values via {step_id.field} syntax; it is for structured data only
- user_query is a separate top-level key for passing a literal query string to the action; it takes priority over the inherited pipeline user_query
- If user_query is omitted, the action automatically inherits the parent pipeline's user_query
- output_step_id is optional and should only be set when the exact inner step id is known; omit it to use automatic result resolution
- When the action returns a list of rows, adopt_internal_source_id is automatically injected for deduplication by downstream WRITE_TO_DB steps
- Valid inside FAN_OUT sub_steps: run an action once per row from an upstream array

Example (simple with upstream data):
{ "id": "enrich_customer", "operation": "RUN_ACTION", "action_id": "abc123", "input": { "customer_name": "{read_step.name}" }, "notes": "Enrich each customer record using the enrichment action" }

Example (with explicit user_query):
{ "id": "search_faqs", "operation": "RUN_ACTION", "action_id": "abc123", "user_query": "list all FAQs about returns", "notes": "Run the FAQ search action with a specific query" }

Example (feeding into WRITE_TO_DB — no output_step_id needed when action has OUTPUT_TABLE):
{ "id": "list_faqs_action", "operation": "RUN_ACTION", "action_id": "abc123", "notes": "Run the list FAQs action; structured rows are returned automatically via formatted_message.data" }

Example (feeding into WRITE_TO_DB — output_step_id used when inner step id is known):
{ "id": "list_faqs_action", "operation": "RUN_ACTION", "action_id": "abc123", "output_step_id": "sortFAQs", "notes": "Return the raw sorted rows from the inner sortFAQs step for WRITE_TO_DB" }

Example (inside FAN_OUT sub_steps):
{
  "id": "fan_out_step",
  "operation": "FAN_OUT",
  "input": "read_records",
  "sub_steps": [
    {
      "id": "run_action_per_row",
      "operation": "RUN_ACTION",
      "action_id": "abc123",
      "input": { "record": "{fan_out_row}" },
      "notes": "Run the processing action on each row"
    }
  ]
}
