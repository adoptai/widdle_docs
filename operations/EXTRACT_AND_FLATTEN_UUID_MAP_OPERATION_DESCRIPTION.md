Define an EXTRACT_AND_FLATTEN_UUID_MAP operation step in a JSON workflow language that transforms a UUID-keyed dictionary into a flat list of dictionaries.

Basic Structure:
{
  "id": string,
  "operation": "EXTRACT_AND_FLATTEN_UUID_MAP",
  "input": string,
  "dict_key": string (optional),
  "uuid_key": string (optional, default: "uuid"),
  "value_key": string (optional, default: "value")
}

Key Features:
- Transforms dictionary with UUID keys into list of dictionaries containing the UUID
- Preserves all nested fields from dictionary values
- Takes output from previous workflow step as input
- Handles both dictionary values and primitive values
- Particularly useful for form schemas and field configurations
- Requires unique operation ID

Examples:

1. Form Schema Fields Example:
Input:
{
  "uuid1": {
    "name": "Full Name",
    "type": "string",
    "required": true
  },
  "uuid2": {
    "name": "Email",
    "type": "email",
    "required": true
  },
  "uuid3": "Simple text field"
}

Operation:
{
  "id": "flattenFormFields",
  "operation": "EXTRACT_AND_FLATTEN_UUID_MAP",
  "input": "getFormSchema"
}

Output:
[
  {
    "uuid": "uuid1",
    "name": "Full Name",
    "type": "string",
    "required": true
  },
  {
    "uuid": "uuid2",
    "name": "Email",
    "type": "email",
    "required": true
  },
  {
    "uuid": "uuid3",
    "value": "Simple text field"
  }
]

2. Configuration Settings Example:
Input:
{
  "setting1": {
    "displayName": "Dark Mode",
    "value": true,
    "category": "appearance"
  },
  "setting2": {
    "displayName": "Font Size",
    "value": 14,
    "category": "text"
  },
  "setting3": false
}

Operation:
{
  "id": "flattenSettings",
  "operation": "EXTRACT_AND_FLATTEN_UUID_MAP",
  "input": "getSettings"
}

Output:
[
  {
    "uuid": "setting1",
    "displayName": "Dark Mode",
    "value": true,
    "category": "appearance"
  },
  {
    "uuid": "setting2",
    "displayName": "Font Size",
    "value": 14,
    "category": "text"
  },
  {
    "uuid": "setting3",
    "value": false
  }
]

Implementation Notes:
- The 'input' field must reference a valid ID from a previous workflow step that returns a dictionary
- Input must be a dictionary; operation will fail if input is null or not a dictionary
- For values that are dictionaries, all key-value pairs are copied to the output object
- For primitive values (strings, numbers, booleans, null), they are stored in a 'value' field
- The original dictionary key is always stored in a 'uuid' field of the output objects
- Output is always an array of objects, even if the input dictionary has only one key