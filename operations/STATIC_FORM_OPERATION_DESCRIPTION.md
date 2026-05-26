Define a STATIC_FORM operation step in a JSON workflow language that emits a fixed, compile-time-validated `form_request` spec as step output. Used by sub-action tools in agent-orchestrated flows to describe structured chat forms without an LLM call.

Basic Structure:
{
  "id": string,
  "operation": "STATIC_FORM",
  "value": object
}

Key Features:
- Zero LLM cost: returns a predefined form spec deterministically
- Compile-time validation: `value` must conform to the shared `form_request` schema (type, schema_version, form_id, fields)
- Same output shape as `PAYLOAD`: validated **dict** in `intermediate_results`; **JSON string** at the sub-action tool boundary (for agent consumption and `enable_form_tools` bridge)
- Same wire format as dynamic forms produced by `PAYLOAD` when emitting `form_request` JSON

Parameters:
- value: Full `form_request` object (required). Must include `type: "form_request"`, `schema_version: 1`, `form_id`, `header_message`, and a non-empty `fields` array. Each field uses `name`, `label`, `componentType` (dropdown, input, datepicker, fileupload), optional `hint`, `required`, and `options` for dropdowns.

Examples:

1. Ticketing main settings form spec (sub-action tool):
{
  "id": "ticketing_main_form_spec",
  "operation": "STATIC_FORM",
  "value": {
    "type": "form_request",
    "schema_version": 1,
    "form_id": "ticketing_main_settings",
    "header_message": "Ticketing Main Settings",
    "fields": [
      {
        "name": "sell_individual_tickets",
        "label": "Will you be selling individual tickets?",
        "hint": "Each ticket admits one attendee",
        "componentType": "dropdown",
        "required": true,
        "options": [
          {"value": "yes", "label": "Yes"},
          {"value": "no", "label": "No"}
        ]
      }
    ]
  }
}

Implementation Notes:
- Step output is stored in `intermediate_results[step_id]` as a **dict** (the validated `form_request` object), matching compile-time type `DICTIONARY` and the same pattern as `PAYLOAD`
- When the sub-action runs as an agent tool, `_resolve_tool_output_string` serializes that dict to a JSON string for LangChain tool output and bridge parsing
- Does not pause the workflow for user input; the parent agent step handles form display via `enable_form_tools: true`
- Invalid `value` fails at compile time (WDL compiler) and at runtime if validation is bypassed
- Prefer `STATIC_FORM` for fixed field lists; use `PAYLOAD` when fields depend on prior context or conversation history
