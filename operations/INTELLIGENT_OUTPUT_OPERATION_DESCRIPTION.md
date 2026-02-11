Define an INTELLIGENT_OUTPUT operation step in a JSON workflow language that uses a Large Language Model (LLM) to intelligently format data into user-facing markdown output.

Basic Structure:
{
  "id": string,
  "operation": "INTELLIGENT_OUTPUT",
  "context_map": object (optional),
  "input": string,
  "instructions": string,
  "table_rows_limit": integer (optional, default: 20)
}

Key Features:
- Uses LLM to automatically determine the best format for presenting data (table, key-value pairs, formatted text, etc.)
- Takes output from a previous workflow step as input
- Supports context variables via context_map to provide additional information to the LLM
- For arrays with more than one row: generates a markdown table showing up to table_rows_limit rows (default: 20), with full data available for download
- For single row or non-array data: LLM determines the most appropriate format
- Always returns markdown format output
- Requires unique operation ID
- Requires LLM configuration to be available

Parameters:
- input: ID of a previous workflow step that contains the data to be formatted (required)
- context_map: Dictionary mapping step IDs to description strings (optional, see format below)
- instructions: Natural language instructions for the LLM on how to format the output (required)
- table_rows_limit: Maximum number of rows to display in the output table for multi-row arrays (optional, default: 20)

## context_map Format

⚠️ IMPORTANT: context_map must use simple string values, NOT nested objects.

✅ CORRECT format:
```json
"context_map": {
  "getUserStats": "User statistics including total actions and success rate",
  "getOrderHistory": "Recent order history for the customer"
}
```

❌ WRONG format (nested objects will cause errors):
```json
"context_map": {
  "getUserStats": {
    "description": "User statistics",
    "render_as": "table"
  }
}
```

The context_map keys must be valid step IDs from previous workflow operations. The string values describe what that step's output contains, helping the LLM understand the context.

Examples:

1. Formatting Product Data:
Input from previous step "getProducts":
[
  {"name": "Laptop", "price": 999.99, "stock": 15},
  {"name": "Mouse", "price": 29.99, "stock": 50},
  {"name": "Keyboard", "price": 79.99, "stock": 30}
]

Operation:
{
  "id": "formatProductList",
  "operation": "INTELLIGENT_OUTPUT",
  "input": "getProducts",
  "instructions": "Format this product list as a clear table showing product names, prices in USD currency format, and stock availability. Include a summary of total products.",
  "table_rows_limit": 20
}

Output:
The LLM will generate a markdown table with appropriate headers and formatting, showing the first 20 rows. Full data is available for download.

2. Formatting Single Object with Context:
Input from previous step "getUserDetails":
{
  "id": "user_123",
  "name": "John Doe",
  "email": "john@example.com",
  "role": "admin",
  "last_login": "2024-01-15T10:30:00Z"
}

Context from previous step "getUserStats":
{
  "total_actions": 150,
  "success_rate": 0.95
}

Operation:
{
  "id": "formatUserProfile",
  "operation": "INTELLIGENT_OUTPUT",
  "input": "getUserDetails",
  "context_map": {
    "getUserStats": "User statistics including total actions and success rate"
  },
  "instructions": "Create a user profile summary in a key-value format. Include user information and mention their activity statistics from the context. Format dates in a human-readable format.",
  "header_message": "User Profile"
}

Output:
The LLM will generate a formatted key-value table or structured text presenting the user information along with the statistics from the context.

3. Formatting Complex Data:
Input from previous step "getOrderSummary":
{
  "order_id": "ORD-12345",
  "customer": {
    "name": "Jane Smith",
    "email": "jane@example.com"
  },
  "items": [
    {"product": "Widget", "quantity": 2, "price": 10.00},
    {"product": "Gadget", "quantity": 1, "price": 25.00}
  ],
  "total": 45.00,
  "status": "completed"
}

Operation:
{
  "id": "formatOrderDetails",
  "operation": "INTELLIGENT_OUTPUT",
  "input": "getOrderSummary",
  "instructions": "Format this order information in a clear, user-friendly way. Show customer details, list the items in a table format, and highlight the total amount and order status.",
}

Output:
The LLM will intelligently format the order data, potentially using a combination of key-value pairs for customer info, a table for items, and formatted text for totals and status.

Implementation Notes:
- The 'input' field must reference a valid ID from a previous workflow step
- Input data can be an array, object, or primitive value
- Nested arrays are not supported (only one level of array is allowed)
- Context variables in context_map must reference valid IDs from previous workflow steps
- Context variables are converted to strings (JSON for objects/arrays) before being passed to the LLM
- The LLM uses the instructions and context to determine the best presentation format
- For multi-row tables, only the first table_rows_limit rows are shown in the output, but full data is prepared for download
- The operation requires LLM configuration to be available; it will return an error if LLM is not configured
- The operation always returns markdown format, which can include tables, key-value pairs, formatted text, or other markdown elements