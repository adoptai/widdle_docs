Define a PROJECT operation step in a JSON workflow language that performs field selection on JSON arrays.

Basic Structure:
{
  "id": string,
  "operation": "PROJECT",
  "input": string,
  "fields": string[] | object
}

Key Features:
- Extracts specified fields from JSON array objects (similar to SQL SELECT)
- Takes output from previous workflow step as input
- Produces new array containing only selected fields
- The id field is a unique operation ID
- Supports two formats for field selection:
  1. Simple list of field names (including dot notation for nested fields)
  2. Object mapping new field names to source field paths

Example with field list:
This example extracts the 'name', 'price', and 'description' fields from each object.
{
  "id": "extractProductFields",
  "operation": "PROJECT",
  "input": "getProductData",
  "fields": ["name", "price", "description"]
}

Example with field mapping:
This example renames fields during projection and supports nested field paths.
{
  "id": "extractUserFields",
  "operation": "PROJECT",
  "input": "getUserData",
  "fields": {
    "userName": "name",
    "userEmail": "contact.email",
    "userPhone": "contact.phone"
  }
}

Notes:
- The 'input' field must reference a valid ID from a previous workflow step
- Input must be a JSON array; operation will fail if input is null or not an array
- For nested fields, use dot notation (e.g., "contact.email")
- When using a field list, the output field name will be the last segment of the path
- When using a field mapping object, the keys are the new field names and the values are the source field paths
- Outputs a new array containing the projected objects with only the selected fields