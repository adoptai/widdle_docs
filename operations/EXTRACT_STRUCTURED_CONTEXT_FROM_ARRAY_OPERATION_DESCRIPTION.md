Define an EXTRACT_STRUCTURED_CONTEXT_FROM_ARRAY operation step in a JSON workflow language that extracts key-value pairs from array elements and stores them in a structured context.

Basic Structure:
{
  "id": string,
  "operation": "EXTRACT_STRUCTURED_CONTEXT_FROM_ARRAY",
  "input": string,
  "key_field": string,
  "value_field": string
}

Key Features:
- Takes an array as input and processes each element
- Extracts specified key and value fields from each array element
- Stores the key-value pairs in a structured context using add_entity_name and _add_entity
- Supports dot notation for accessing nested fields
- Requires unique operation ID

Example:
Input:
[
  {
    "id": "user_1",
    "details": {
      "name": "John Doe",
      "role": "admin"
    }
  },
  {
    "id": "user_2",
    "details": {
      "name": "Jane Smith",
      "role": "user"
    }
  }
]

Operation:
{
  "id": "extractUserContext",
  "operation": "EXTRACT_STRUCTURED_CONTEXT_FROM_ARRAY",
  "input": "getUsers",
  "key_field": "id",
  "value_field": "details.name"
}

This operation will store the following key-value pairs in the structured context:
- "user_1" -> "John Doe"
- "user_2" -> "Jane Smith"

Notes:
- The input must be an array; operation will fail if input is null or not an array
- Both key_field and value_field must be specified
- Uses dot notation for accessing nested fields (e.g., "details.name")
- Keys must be unique; if duplicate keys are found, the last one processed will be used