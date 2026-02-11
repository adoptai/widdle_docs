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
- **extract_all: true (DEFAULT)** - Wraps output in an array. If your filter returns `[item1, item2]`, you get `[[item1, item2]]` (double-wrapped!)
- **extract_all: false** - Returns the first item directly. Use this when you want a single object, not an array.

**RECOMMENDATION**: Always explicitly set `extract_all: false` unless you specifically need array output. This prevents the common issue of unexpected array wrapping that causes downstream EXTRACT operations to fail.

Key Features:
- Takes output from previous workflow steps as input
- Uses a JQ filter to apply a JQ operation to the input(s)
- The filter is a valid JQ expression that is applied to the input
- Single input mode: Use "input" to reference one previous step
- Multiple inputs mode: Use "inputs" array to reference multiple steps. The inputs are combined into an object where each key is the step id and the value is the step output. Example: {"step1": [...], "step2": [...]}
- Inputs can be a JSON object or an array. Outputs are also either a JSON object or an array.

Example 1: Single object extraction (RECOMMENDED pattern)
{
  "id": "extractFirstUser",
  "operation": "JQ_FILTER",
  "input": "getUsers",
  "filter": ".[0]",
  "extract_all": false
}
Output: The first user object directly (not wrapped in array)

Example 2: Multiple inputs (concatenating two lists)
{
  "id": "combineLists",
  "operation": "JQ_FILTER",
  "inputs": ["getUsers", "getAdmins"],
  "filter": ".getUsers + .getAdmins",
  "extract_all": true
}
Output: Combined array of users and admins

Example 3: Transform and extract single item
{
  "id": "getCampaignDetails",
  "operation": "JQ_FILTER",
  "input": "searchResults",
  "filter": ".Items[0] | {id: .ID, name: .Name}",
  "extract_all": false
}
Output: Single campaign object {id: "123", name: "Campaign Name"}

Common Pitfall - Array Wrapping:
If you omit extract_all (defaults to true), your subsequent EXTRACT operation may fail with:
"Input is not a JSON object. It is a <class 'list'>. Extraction requires a JSON object."

Fix: Add `"extract_all": false` to your JQ_FILTER.

Use this operation to perform arbitrary transformation on the output of previous steps, before rendering it as output or as input to a subsequent step.