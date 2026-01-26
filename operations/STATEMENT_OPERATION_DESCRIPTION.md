Define a STATEMENT operation step in a JSON workflow language that provides a human-readable description or title for the workflow action.

Basic Structure:
{
  "id": string,
  "operation": "STATEMENT",
  "statement": string
}

Description:
- Provides a human-readable statement describing what the action does
- Used as the title or description shown to users in the UI
- This is a metadata operation that doesn't perform any data processing
- Should describe the action's purpose in clear, user-friendly language

Key Features:
- Defines the action's display name/description
- Does not modify or process any data
- Typically placed at the beginning of a workflow
- The statement should be concise and action-oriented
- Requires unique operation ID

Parameters:
- id: Unique identifier for this step (commonly "statement")
- operation: Must be "STATEMENT"
- statement: Human-readable description of what the workflow does

Examples:

1. Simple Action Statement
```json
{
  "id": "statement",
  "operation": "STATEMENT",
  "statement": "Get employee details by ID"
}
```

2. Action with Descriptive Statement
```json
{
  "id": "statement",
  "operation": "STATEMENT",
  "statement": "Create a new support ticket and assign it to the appropriate team"
}
```

3. Statement in Context of a Workflow
```json
[
  {
    "id": "statement",
    "operation": "STATEMENT",
    "statement": "List all pending orders for the selected customer"
  },
  {
    "id": "required_inputs",
    "operation": "REQUIRED_INPUTS",
    "required_inputs": ["customer_id"]
  },
  {
    "id": "getOrders",
    "operation": "REST",
    "url": "/api/customers/{{workflow_parameters.customer_id}}/orders",
    "method": "GET",
    "query_params": {
      "status": "pending"
    }
  }
]
```

Note: When generating workflows, do not edit existing STATEMENT operations. Instead, repeat them as-is from the source action definition.
