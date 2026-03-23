Define an ESCALATE operation step in a JSON workflow language. ESCALATE acts as a **validation gate**: it checks confidence AND required fields, then either pauses for human review or lets the pipeline continue.

Basic Structure:
{
  "id": string,
  "operation": "ESCALATE",
  "input": string (reference to previous step whose output should be validated/reviewed),
  "question": string (the question or instruction shown to the human reviewer if escalation triggers),
  "required_fields": array of strings (dot-paths to fields that must be non-null/non-empty in the input data),
  "confidence_score": string or number (template variable or literal; 0.0-1.0),
  "confidence_threshold": number (optional; default 0.75),
  "context": object (optional; additional context for the reviewer),
  "escalation_type": string (optional; default "low_confidence"),
  "display_label": string (optional; human-readable label for the escalation),
  "priority": string (optional; "low", "normal", "high", "critical"; default "normal"),
  "tags": array (optional; string tags for categorization)
}

Key Behavior:
- **Validation gate mode** (when required_fields is provided): Checks BOTH confidence AND required fields.
  - If confidence >= threshold AND all required fields are present → pipeline CONTINUES (no pause, no escalation).
  - If confidence < threshold OR any required field is null/empty → creates escalation and PAUSES.
- **Legacy mode** (no required_fields): Always creates an escalation and pauses (backward-compatible).
- When required_fields triggers escalation, the question is auto-enriched with the list of missing fields.
- Priority is auto-bumped to "high" when fields are missing (even if confidence was above threshold).
- escalation_type is set to "missing_required_fields" when null/empty fields are the trigger.

IMPORTANT: When generating extraction pipelines, ALWAYS use the validation gate pattern. Place ESCALATE AFTER the extraction PROMPT step and BEFORE WRITE_TO_DB, with required_fields listing the fields the user expects. This eliminates the need for a separate CONDITION step — ESCALATE handles both confidence and field validation.

Pipeline pattern (recommended):
  PARSE_DOCUMENT → PROMPT (extract) → AGENTIC_SEARCH → PROMPT (fill) → ESCALATE (validate) → WRITE_TO_DB

required_fields uses dot-path notation into the input step's output:
  - "extracted_fields.employee_name" checks input["extracted_fields"]["employee_name"]
  - "confidence_score" checks input["confidence_score"]

Examples:

1. Validation Gate with Required Fields (RECOMMENDED for extraction pipelines):
{
  "id": "validateExtraction",
  "operation": "ESCALATE",
  "input": "fillDocument",
  "question": "Extraction needs human review.",
  "required_fields": [
    "extracted_fields.employee_name",
    "extracted_fields.employer_name",
    "extracted_fields.total_gross_salary",
    "extracted_fields.tax_deducted_tds",
    "extracted_fields.pan_number"
  ],
  "confidence_score": "{{fillDocument.confidence_score}}",
  "confidence_threshold": 0.75,
  "display_label": "Extraction Review - {{parseDoc.filename}}",
  "priority": "normal",
  "notes": "Validates confidence and required fields; only pauses if something is wrong"
}

2. Low Confidence Only (legacy — no field validation):
{
  "id": "escalateForReview",
  "operation": "ESCALATE",
  "input": "fillDocument",
  "question": "The document fill has low confidence. Please review.",
  "confidence_score": "{{fillDocument.confidence_score}}",
  "confidence_threshold": 0.75,
  "priority": "high"
}

3. Conflict Resolution:
{
  "id": "resolveConflict",
  "operation": "ESCALATE",
  "input": "mergedResults",
  "question": "Cross-document conflict detected.",
  "required_fields": ["line_total", "summary_total"],
  "escalation_type": "conflict",
  "confidence_score": 0.3,
  "confidence_threshold": 0.75,
  "priority": "critical",
  "tags": ["conflict", "financial"]
}

Data Quality Flags (Adversarial Audit Pattern):
When the input step contains a `data_quality_flags` array (typically from an adversarial audit PROMPT step), ESCALATE automatically:
- Adds a "Data Quality Issues" JSON preview to the HITL dashboard context
- Appends a summary ("Audit found N data quality issue(s).") to the escalation question
- No configuration needed — if `data_quality_flags` is present and non-empty, it is surfaced

Recommended adversarial audit pipeline pattern:
  PARSE_DOCUMENT → PROMPT (extract only, no confidence) → JQ_FILTER (merge extracted + source text) → PROMPT (adversarial audit: cross-check, flag issues, score confidence) → ESCALATE (gate on audit confidence + flags) → WRITE_TO_DB

Each `data_quality_flags` entry should have: `field` (string), `issue` (e.g. "ocr_error", "cross_check_mismatch", "incomplete_data", "draft_document", "invalid_value"), `severity` ("low", "medium", "high", "critical"), and `details` (human-readable explanation).

Implementation Notes:
- question is required when required_fields is not provided; when required_fields is set, a question is auto-generated from validation failures if not explicitly given
- The input field references a previous step; its output dict is included in the escalation context and validated against required_fields
- ESCALATE writes directly to the HITL escalation store (db_org_hitl_escalations and db_org_hitl_conversation_messages)
- The result dict includes: escalation_id, conversation_id, action_instance_id, pipeline_id, pipeline_run_id, workflow_id, missing_fields
- On re-run after human resolution, AGENTIC_SEARCH steps with include_hitl_context=true will automatically inject the human corrections
- Unlike HUMAN_IN_LOOP (two-phase pause/resume with LLM gating), ESCALATE uses full-rerun semantics
- On re-run after human resolution, ESCALATE checks intermediate_results for prior corrections (injected by assign_user_response_to_input_block). If corrections are found, they are merged into extracted_fields before validation. If all required_fields are now present, ESCALATE passes through with status "human_corrected" (confidence gate is skipped — the human reviewed).
- Fallback: if corrections are not in intermediate_results, ESCALATE queries the DB for the most recent resolved escalation for the same pipeline+step and applies corrections from the resolution column.
