Defines a CONDITION operation in a JSON workflow language that evaluates conditions and determines the next step based on the result.

Basic Structure:
{
  "id": string,
  "operation": "CONDITION",
  "input": string,
  "compound": string ("AND" or "OR"),
  "clauses": [
    {
      "field": string,
      "operator": string,
      "value": any
    }
  ],
  "then": string,
  "else": string,
  "type_hint": object (optional, default: {})
}

Key Features:
- Evaluates conditions on JSON objects
- Supports compound conditions using AND/OR logic
- Takes output from previous workflow step as input
- Requires unique operation ID
- Determines the next step based on the evaluation result
- The 'then' field specifies the next step if the condition is true
- The 'else' field specifies the next step if the condition is false

Supported operators for clauses:

Numeric / equality (native jq infix):
  ==  !=  >  <  >=  <=

String operators — default is case-insensitive. Append _CS for explicit case-sensitive matching:
  CONTAINS / CONTAINS_CS         — field contains substring
  NOT_CONTAINS / NOT_CONTAINS_CS — field does NOT contain substring
  STARTS_WITH / STARTS_WITH_CS   — field begins with value
  ENDS_WITH / ENDS_WITH_CS       — field ends with value
  MATCHES / MATCHES_CS           — field matches regex pattern
  NOT_MATCHES / NOT_MATCHES_CS   — field does NOT match regex pattern

IMPORTANT: Only the operators listed above are valid. Do NOT invent operators (e.g. "CONTAINS_TOKEN", "CONTAINS_I", "CONTAINS_WORD"). Always prefer the base form (e.g. CONTAINS) which is case-insensitive. Only use _CS when an exact case-sensitive match is explicitly required.

Examples:
1. Conditional Step Execution:
Input:
{
  "order": {
    "total_items": 30,
    "status": "pending"
  }
}

Operation:
{
  "id": "checkOrderItems",
  "operation": "CONDITION",
  "input": "getOrder",
  "compound": "AND",
  "clauses": [
    {
      "field": "total_items",
      "operator": ">",
      "value": 25
    }
  ],
  "then": "processLargeOrder",
  "else": "processSmallOrder"
}

If the condition is true, the next step to be executed is the one with id "processLargeOrder", otherwise it is "processSmallOrder".