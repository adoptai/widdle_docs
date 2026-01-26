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

Key Features:
- Takes output from previous workflow steps as input
- Uses a JQ filter to apply a JQ operation to the input(s)
- The filter is a valid JQ expression that is applied to the input
- Single input mode: Use "input" to reference one previous step
- Multiple inputs mode: Use "inputs" array to reference multiple steps. The inputs are combined into an object where each key is the step id and the value is the step output. Example: {"step1": [...], "step2": [...]}
- Inputs can be a JSON object or an array. Outputs are also either a JSON object or an array.

Example with multiple inputs (concatenating two lists):
{
  "id": "combineLists",
  "operation": "JQ_FILTER",
  "inputs": ["getUsers", "getAdmins"],
  "filter": ".getUsers + .getAdmins"
}

Use this operation to perform arbitrary transformation on the output of previous steps, before rendering it as output or as input to a subsequent step.
Prefer extract_all to be set to false so that it will remove the need to treat the output as an array in subsequent steps. When the user intent specifies that they want a list of items, then set extract_all to true.
Note with extra care that if extract_all is missing, it will default to true. extract_all forces calling .all on the JQ expression. When you don't want that, set extract_all to false and this will return the first item in the array.