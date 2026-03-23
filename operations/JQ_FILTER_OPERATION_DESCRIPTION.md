A JQ filter that allows you to apply a JQ operation to the output of previous steps.

Basic Structure:
{
  "operation": "JQ_FILTER",
  "id": string,
  "input": string (optional - use this OR inputs, not both),
  "inputs": string[] (optional - use this OR input, not both),
  "filter": string,
  "type_hint": string (optional),
  "extract_all": boolean (optional) (default: true if this field is not provided)
}

⚠️ CRITICAL: Output Behavior
- The output of JQ_FILTER is wrapped in a `result` key: `{"result": <your_filter_output>}`
- If your filter returns `[{...}]`, the actual output is `{"result": [{...}]}`
- Subsequent operations receive the `result` value, not the raw wrapper

⚠️ CRITICAL: extract_all Parameter (DEFAULT: true)

The choice is based on **how many results your JQ filter emits**, NOT on whether you want an object or an array output.

- **extract_all: false** → calls `.first()` — returns the **first (and usually only) JQ result** directly.
  Use this when your filter emits ONE result, even if that result is itself an array.
  Example: filter `.emails` emits one result (the whole array `[{...}, {...}]`). Use `extract_all: false` → output is `[{...}, {...}]` ✓

- **extract_all: true (DEFAULT)** → calls `.all()` — collects **every individual JQ output** into a list.
  Use this ONLY when your filter emits MULTIPLE individual results (e.g. `.emails[]` emits one object per email).
  Example: filter `.emails[]` emits N results. Use `extract_all: true` → output is `[{email1}, {email2}]` ✓

⚠️ COMMON MISTAKE: Using `extract_all: true` when your filter returns a single array (e.g. `.emails` which returns `[{...}]`).
`.all()` wraps that one result in another list → `[[{...}]]` (double-nested). This breaks downstream WRITE_TO_DB and EXTRACT steps.

**RULE OF THUMB**:
- Filter ends with `[]` or generates multiple outputs → `extract_all: true`
- Filter returns a single object or single array → `extract_all: false`

Key Features:
- Takes output from previous workflow steps as input
- Uses a JQ filter to apply a JQ operation to the input(s)
- The filter is a valid JQ expression that is applied to the input
- Single input mode: Use "input" to reference one previous step
- Multiple inputs mode: Use "inputs" array to reference multiple steps. The inputs are combined into an object where each key is the step id and the value is the step output. Example: {"step1": [...], "step2": [...]}
- Inputs can be a JSON object or an array. Outputs are also either a JSON object or an array.

⚠️ CRITICAL: Variable binding in map() — NEVER use `input` as a variable

In JQ, `input` is a built-in function that reads the next value from stdin—it is NOT a reference to the current object or its parent. Using `input.field_name` inside a map() or any expression causes a runtime "break" or "null input" error when stdin is exhausted.

To access parent object fields within a nested expression like map(), capture the parent first using the JQ variable binding syntax: `. as $parent | .child_array | map({field: $parent.field, ...})`.

Example — expand a nested array while carrying forward parent fields:
`. as $parent | if (.items | length) > 0 then .items | map({id: .id, parent_name: $parent.name}) else [] end`

NEVER write: `.items | map({parent_name: input.name})` — always use `. as $parent | .items | map({parent_name: $parent.name})`.

⚠️ CRITICAL: inputs (plural) + map — capture metadata before map()

When a JQ_FILTER uses "inputs" to merge metadata (e.g. prepare_email_for_prompt) with a PROMPT array output (e.g. extract_freight_rates.extracted_rates) and maps over that array, inside map() the context "." is each array element—NOT the root. So `.prepare_email_for_prompt.mail_id` inside map() is null because rate objects have no such key.

ALWAYS capture metadata before the map: use `.prepare_email_for_prompt as $meta |` (or `.step_id as $meta |`) then inside map() use `$meta.mail_id`, `$meta.subject`, etc.

Example — merge metadata with extracted rates:
`if (.extract_freight_rates.extracted_rates | type) == "array" and (.extract_freight_rates.extracted_rates | length) > 0 then .prepare_email_for_prompt as $meta | .extract_freight_rates.extracted_rates | map({mail_id: $meta.mail_id, subject: $meta.subject, from: $meta.from, from_name: $meta.from_name, received_datetime: $meta.received_datetime, rate_data: .}) else [] end`

NEVER write: `.extract_freight_rates.extracted_rates | map({mail_id: .prepare_email_for_prompt.mail_id, ...})` — that yields null metadata.

Example 1: Single object extraction (RECOMMENDED pattern)
{
  "id": "extractFirstUser",
  "operation": "JQ_FILTER",
  "input": "getUsers",
  "filter": ".[0]",
  "extract_all": false
}
Output: The first user object directly (not wrapped in array)

Example 2: Multiple inputs (concatenating two lists) — filter emits ONE result → extract_all: false
{
  "id": "combineLists",
  "operation": "JQ_FILTER",
  "inputs": ["getUsers", "getAdmins"],
  "filter": ".getUsers + .getAdmins",
  "extract_all": false
}
Output: Combined array of users and admins (e.g. [{user1}, {user2}, {admin1}])
Note: extract_all: false is correct here because `.getUsers + .getAdmins` emits ONE result (the merged array).
Using extract_all: true would double-wrap it: [[{user1}, {user2}, {admin1}]] ❌

Example 3: Transform and extract single item
{
  "id": "getCampaignDetails",
  "operation": "JQ_FILTER",
  "input": "searchResults",
  "filter": ".Items[0] | {id: .ID, name: .Name}",
  "extract_all": false
}
Output: Single campaign object {id: "123", name: "Campaign Name"}

Example 4: Extract items array from a response object — filter emits ONE result → extract_all: false
{
  "id": "extractEmails",
  "operation": "JQ_FILTER",
  "input": "fetchOutlookEmails",
  "filter": ".emails",
  "extract_all": false
}
Output: [{mail_id: "...", subject: "..."}, ...] — the emails array passed through directly ✓
Note: extract_all: true here would give [[{...}]] (double-nested) ❌

Example 5: Unpack array items individually — filter emits MULTIPLE results → extract_all: true
{
  "id": "unpackEmails",
  "operation": "JQ_FILTER",
  "input": "fetchOutlookEmails",
  "filter": ".emails[]",
  "extract_all": true
}
Output: [{mail_id: "...", subject: "..."}, {mail_id: "...", subject: "..."}] — flat list of individual email objects ✓
Note: Use this pattern when feeding into FAN_OUT or when you want each item as a separate element.

Use this operation to perform arbitrary transformation on the output of previous steps, before rendering it as output or as input to a subsequent step.