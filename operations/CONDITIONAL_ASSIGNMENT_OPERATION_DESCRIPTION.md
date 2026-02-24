Define a CONDITIONAL_ASSIGNMENT operation step in a JSON workflow language that selects between two previous operation results based on condition evaluation.

Basic Structure:
{
  "id": string,
  "operation": "CONDITIONAL_ASSIGNMENT",
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
  "default": string,
  "type_hint": object (optional, default: {})
}

Key Features:
- Evaluates conditions on a JSON object and returns one of two previous operation results
- Takes output from previous workflow steps as inputs
- Supports compound conditions using AND/OR logic
- The 'then' field specifies the operation result to use if conditions are true
- The 'default' field specifies the operation result to use if conditions are false
- Requires unique operation ID
- Passes through the selected operation result without modifying it

Examples:
Basic Conditional Assignment:
Previous Results:
{
  "userInfo": {
    "membership": "premium",
    "region": "US"
  },
  "premiumContent": {
    "videos": ["video1.mp4", "video2.mp4"],
    "articles": ["article1.html", "article2.html"]
  },
  "standardContent": {
    "videos": ["video1.mp4"],
    "articles": ["article1.html"]
  }
}

Operation:
{
  "id": "selectContent",
  "operation": "CONDITIONAL_ASSIGNMENT",
  "input": "userInfo",
  "compound": "AND",
  "clauses": [
    {
      "field": "membership",
      "operator": "==",
      "value": "premium"
    }
  ],
  "then": "premiumContent",
  "default": "standardContent"
}

Output:
If userInfo.membership is "premium", the output will be:
{
  "videos": ["video1.mp4", "video2.mp4"],
  "articles": ["article1.html", "article2.html"]
}

Otherwise, the output will be:
{
  "videos": ["video1.mp4"],
  "articles": ["article1.html"]
}

2. No Clauses Example (True if input exists):
Previous Results:
{
  "fullAccess": {
    "level": "complete",
    "features": ["dashboard", "reports", "admin"]
  },
  "limitedAccess": {
    "level": "basic",
    "features": ["dashboard"]
  }
}

Operation:
{
  "id": "determineAccess",
  "operation": "CONDITIONAL_ASSIGNMENT",
  "input": "fullAccess",
  "then": "fullAccess",
  "default": "limitedAccess"
}

Output:
Since no clauses are provided, the condition is 'if input exists', so the output will be:
{
  "level": "complete",
  "features": ["dashboard", "reports", "admin"]
}
If the input is not found, the output will be:
{
  "level": "basic",
  "features": ["dashboard"]
}

Notes:
- The 'input' field must reference a valid ID from a previous workflow step
- The 'then' and 'default' fields must reference valid IDs from previous workflow steps
- The operation does not modify the selected result, it simply passes it through
- If 'input' is available but no 'clauses' are provided, the 'then' result is always selected
- If the 'input' object is not found or is invalid, the 'default' result is selected
- Condition evaluation uses JQ-style querying for field values and comparisons
- Supported operators include ==, !=, >, <, >=, and <=