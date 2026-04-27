Define a TRANSLATE operation step in a JSON workflow language that translates human-facing strings from one language to another. The input can be a single form dict, an array of form dicts, or an array of plain strings; the output always matches the input shape.

Basic Structure:
{
  "id": string,
  "operation": "TRANSLATE",
  "input": string (step ID whose output is the payload: list[str] | list[dict] | dict),
  "target_lang": string (IETF BCP 47 language tag — e.g. "es", "fr", "zh-CN"; default "es"),
  "source_lang": string (IETF BCP 47 language tag; default "en"),
  "preferred_llm": string (optional LLMName value, default "openai/gpt-oss-20b"),
  "translatable_keys": list[string] (optional — overrides default translatable keys; applies to dict / list[dict] inputs only),
  "skip_keys": list[string] (optional — overrides default skipped keys; applies to dict / list[dict] inputs only),
  "context_skip": object (optional — overrides default context-aware skip rules; shape: {"<parent_key>": ["<child_key>", ...]})
}

Description:
TRANSLATE takes a payload from a previous step and returns a deep copy with translatable strings replaced. The operation auto-detects the payload shape from the first element of the input and returns the same shape in the same order.

For dict / list[dict] inputs (Centuri form payloads), extraction is schema-agnostic — it walks the entire JSON tree dynamically and translates values under translatable keys (label, text, notes), leaving machine-code fields (id, value, type, rules, logic, etc.) untouched. No hardcoded paths.

For list[str] inputs, each string is translated and the output preserves order and multiplicity (duplicates in the input translate once but are repeated in the output at every original position).

Input shapes:
- list[str]  -> list[str]   Array of plain strings, translated in order.
- list[dict] -> list[dict]  Array of form dicts. All strings across the array are batched into a single engine call for efficiency; each form is reconstructed independently with its slice of the translations.
- dict       -> dict        Single form dict (legacy path).

Translation Engine:
Default model is LLMName.GROQ_GPT_OSS_20B (Groq's openai-gpt-oss-20b, ~900 tok/s on Groq LPU — benchmarked as the best latency/quality trade-off for Centuri form payloads). Routes through ConfigInstance.create_llm so the same env vars the rest of ProjectA3 uses drive the underlying LLM — no translate-specific env vars. Sends all strings for a given step in a single LLM call as a JSON translation task. Handles thinking blocks, markdown code fences, and whitespace-wrapped JSON. On parse failure or API error, the step returns an error rather than silently returning untranslated text.

Environment variables (inherited from the standard ProjectA3 LLM routing):
- Gateway path (production):  LLM_GATEWAY_ENABLED=true, LLM_GATEWAY_API_KEY, LLM_GATEWAY_URL
- Direct Groq path (non-gateway): GROQ_API_KEY

If no usable credentials are configured at the settings layer, ConfigInstance.create_llm raises and the step fails. There is no automatic fallback; a single-engine architecture keeps the operation simple and caching (if needed) is handled at the frontend layer.

Output:
{
  "translated_payload": <same shape as input — list[str], list[dict], or dict>,
  "translation_map": {
    "<opaque_key>": {
      "original": string,
      "translated": string
    },
    ...
  }
}

The translation_map key format depends on the input shape:
- For list[str]  : keys are string positions ("0", "1", "2", ...).
- For list[dict] : keys are "<index>.<dot_path>" (e.g. "0.sections.1.label").
- For dict       : keys are dot-paths (e.g. "sections.0.fields.2.options.1.text").

Key Features:
- Schema-agnostic recursive extraction — works on any form structure without hardcoded paths.
- Context-aware skipping: "text" inside "answers" is treated as a machine code (e.g. "YES") and left untouched, while "text" inside "options" is translated (e.g. "North").
- Default translatable keys (Centuri-tuned, overridable per step): label, text, notes.
- Default skipped keys (Centuri-tuned, overridable per step): id, value, type, rules, logic, signatures, media, url, buildTime, buildError, formId, area, workType, status, isReadOnly, canCopy, canClose, canDelete, selectedTraining, trainingInfo.
- Per-customer reusability: the defaults above match Centuri payloads, but every rule can be overridden per step via translatable_keys / skip_keys / context_skip without code changes.
- Deduplication: identical strings across the payload are translated only once per engine call; dedupe preserves first-seen order so prompts are stable across runs.
- Non-destructive: always returns a deep copy; the original payload is never modified.
- Array batching: for list[dict] input, every form's strings are sent in a single engine call — N forms cost one HTTP round-trip, not N.

Parameters:
- input (required): Step ID whose output is the payload (list[str] | list[dict] | dict).
- target_lang (optional, default "es"): IETF BCP 47 language tag (e.g. "es", "fr", "de", "zh-CN", "pt-BR").
- source_lang (optional, default "en"): IETF BCP 47 language tag.
- preferred_llm (optional, default "openai/gpt-oss-20b"): LLMName value-string to override the model. Must resolve to a valid LLMName enum entry (e.g. "openai/gpt-oss-120b"); invalid values fall back to the default with a warning.
- translatable_keys (optional): list of key names whose string values should be translated. Overrides the Centuri-tuned default set. Applies to dict and list[dict] inputs only.
- skip_keys (optional): list of key names whose subtrees should be skipped entirely during extraction (machine codes, metadata, system fields). Overrides the Centuri-tuned default set. Applies to dict and list[dict] inputs only.
- context_skip (optional): object mapping a parent key to a list of child keys that should not be translated inside that parent. Example: {"answers": ["text", "url"]}. Overrides the Centuri-tuned default. Applies to dict and list[dict] inputs only.

Examples:

1. Translate a single form to Spanish:
{
  "id": "translateForm",
  "operation": "TRANSLATE",
  "input": "fetchForm"
}

2. Translate an array of forms to Spanish (batched in one engine call):
{
  "id": "translateForms",
  "operation": "TRANSLATE",
  "input": "fetchForms",
  "target_lang": "es"
}

3. Translate an array of safety phrases to French:
{
  "id": "translatePhrases",
  "operation": "TRANSLATE",
  "input": "safetyPhrases",
  "target_lang": "fr",
  "source_lang": "en"
}

4. Translate a non-Centuri form with customer-specific keys (custom schema):
{
  "id": "translateCustomerForm",
  "operation": "TRANSLATE",
  "input": "fetchForm",
  "target_lang": "fr",
  "translatable_keys": ["title", "description", "placeholder"],
  "skip_keys": ["uuid", "schemaVersion", "tenantId"],
  "context_skip": {"validators": ["message"]}
}

5. Chain with a REST call and output:
[
  {
    "id": "fetchForm",
    "operation": "REST",
    "method": "GET",
    "url": "https://api.centuri.com/forms/{{formId}}"
  },
  {
    "id": "translateForm",
    "operation": "TRANSLATE",
    "input": "fetchForm",
    "target_lang": "es"
  },
  {
    "id": "output",
    "operation": "OUTPUT_TEXT",
    "input": "translateForm",
    "template": "Form translated successfully. {{translateForm.translation_map | length}} strings translated."
  }
]

Implementation Notes:
- input is required and must reference a step whose output is a list[str], list[dict], or dict.
- target_lang and source_lang accept IETF BCP 47 language tags (plain ISO 639-1 codes like "es" work, and regional variants like "zh-CN" or "pt-BR" work too).
- The translation_map in the output provides a full audit trail mapping each key to its original and translated value.
- The translated_payload can be passed directly to downstream steps (e.g. REST PUT to save the translated form, FILE_EXPORT, or OUTPUT_TEXT).
- Mixed-type lists (e.g. one string and one dict in the same array) are rejected with an error.
- translatable_keys / skip_keys / context_skip are how other customers reuse this operation — supply your own key lists per step and the extractor adapts to your form schema without any code changes.
