Define a PAYLOAD operation step in a JSON workflow language that uses a language model to generate structured JSON output conforming to a specified schema.

Basic Structure:
{
  "id": string,
  "operation": "PAYLOAD",
  "instructions": string,
  "json_schema": object,
  "input": string (optional),
  "inputs": string[] (optional),
  "max_retries": integer (optional, default: 2),
  "preferred_llm": string (optional)
}

Key Features:
- Generates structured JSON output that strictly conforms to a provided JSON schema
- Uses natural language instructions to guide the language model
- Automatically validates output against the schema and retries with error feedback if validation fails
- Takes output from previous workflow steps as optional context
- Supports both single input and multiple inputs for richer context
- Requires unique operation ID

Parameters:
- instructions: Natural language instructions describing what data to generate and how (required, max 200 words)
- json_schema: A valid JSON Schema (draft-07 or later) that the output must conform to (required)
- input: ID of a previous workflow step whose output provides context (optional)
- inputs: List of IDs from previous workflow steps whose outputs provide context (optional)
- max_retries: Maximum number of retry attempts if schema validation fails (optional, default: 2)
- preferred_llm: LLM model to use for generation (optional, default: uses config heavy_llm). Options include: "claude-haiku-4-5", "claude-sonnet-4-5", "openai-new/gpt-5", etc.

Examples:

1. Extract Structured Data from Text (with nested arrays):
{
  "id": "extractInvoiceData",
  "operation": "PAYLOAD",
  "instructions": "Parse the invoice text and extract the vendor name, invoice number, line items with descriptions and amounts, and total amount.",
  "input": "getInvoiceText",
  "json_schema": {
    "type": "object",
    "properties": {
      "vendor_name": {"type": "string"},
      "invoice_number": {"type": "string"},
      "line_items": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "description": {"type": "string"},
            "amount": {"type": "number"}
          },
          "required": ["description", "amount"]
        }
      },
      "total_amount": {"type": "number"}
    },
    "required": ["vendor_name", "invoice_number", "line_items", "total_amount"]
  },
  "max_retries": 3
}

2. Transform Data with Multiple Inputs:
{
  "id": "mergeUserAndPermissions",
  "operation": "PAYLOAD",
  "instructions": "Combine the user details and their permission settings into a unified access control object.",
  "inputs": ["getUserDetails", "getPermissions"],
  "json_schema": {
    "type": "object",
    "properties": {
      "user_id": {"type": "string"},
      "display_name": {"type": "string"},
      "permissions": {
        "type": "object",
        "properties": {
          "can_read": {"type": "boolean"},
          "can_write": {"type": "boolean"},
          "can_admin": {"type": "boolean"}
        }
      }
    },
    "required": ["user_id", "permissions"]
  }
}

Implementation Notes:
- The 'json_schema' field must be a valid JSON Schema object
- The schema is always included in the LLM prompt to guide generation
- Conversation history is always passed to the LLM to provide context about the user's request
- If 'input' or 'inputs' are provided, they supplement the conversation history with structured data from previous steps
- If 'input' or 'inputs' are not provided, the operation relies solely on conversation history as context
- When validation fails, the LLM is prompted again with the specific validation errors to self-correct
- After max_retries attempts, the operation returns an error if the output still doesn't conform to the schema
- Instructions are limited to 200 words maximum for performance reasons
- The output of this operation is always the validated JSON object, ready for use by subsequent steps
- Use this operation when you need guaranteed schema conformance; use PROMPT for free-form text/table generation