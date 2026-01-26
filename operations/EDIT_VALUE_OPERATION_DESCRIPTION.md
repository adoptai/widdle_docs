Define an EDIT_VALUE operation step in a JSON workflow language that modifies JSON objects by adding, updating, or removing fields or array elements.

Basic Structure:
{
  "id": "string",
  "operation": "EDIT_VALUE",
  "input": "string",
  "sub_operation": "string",
  "field": "string_dot_notation (optional)",
  "value": "any (context-dependent)",
  "type_hint": "object (optional)",
  "transformations": "object (optional)"
}

Key Features:
- Modifies existing JSON objects or arrays with a rich set of sub-operations.
- Takes the output from a previous workflow step as its input.
- Uses a sub_operation parameter to specify the exact type of modification.
- Can target nested data within the input object using a dot-notation field path.
- The value parameter's meaning depends on the chosen sub_operation.
- Supports advanced parameter substitution with optional type_hint.
- Returns the entire modified input object as its output.

Supported Sub-Operations:
1. ASSIGN - Replace the value at the specified field path
2. ARRAY_APPEND - Add elements to the end of an array
3. ARRAY_DELETE - Remove elements from an array based on matching criteria
4. ARRAY_DICT_SUBSET - Create a new array with only specified keys from each dictionary
5. ARRAY_DICT_RENDER - Transform array of dictionaries using a template structure. Optionally you can provide transformations which apply methods to the field values.
6. DICT_SUBSET - Create a new dictionary with only specified keys
7. DICT_MERGE - Combine two dictionaries, with value fields overriding input fields
8. ARRAY_DICT_TO_ARRAY - Convert an array of dictionaries into a flat array of values based on specified keys
9. ARRAY_TO_ARRAY_DICT - Convert an array of values into an array of dictionaries with specified key
10. DICT_TO_ARRAY_DICT - Convert a dictionary into an array of key-value pairs
11. CONVERT_TO_ARRAY: Converts a string into an array by splitting it on a separator.
12. STRING_TO_INTEGER: Converts a string value to an integer.
13. STRING_TO_FLOAT: Converts a string value to a float.
14. STRING_TO_BOOLEAN: Converts a string value (e.g., "true", "false") to a boolean.

Examples:

1. ASSIGN Example - Replace a value:
Input:
{
  "product": {
    "name": "Widget",
    "price": 10.99,
    "inStock": true
  }
}

Operation:
{
  "id": "updatePrice",
  "operation": "EDIT_VALUE",
  "input": "getProduct",
  "field": "product.price",
  "sub_operation": "ASSIGN",
  "value": 12.99
}

Output:
{
  "product": {
    "name": "Widget",
    "price": 12.99,
    "inStock": true
  }
}

2. ARRAY_APPEND Example - Add an item to a list:
Input:
{
  "cart": {
    "items": [
      {"id": "prod-1", "quantity": 2},
      {"id": "prod-2", "quantity": 1}
    ],
    "total": 35.98
  }
}

Operation:
{
  "id": "addToCart",
  "operation": "EDIT_VALUE",
  "input": "getCart",
  "field": "cart.items",
  "sub_operation": "ARRAY_APPEND",
  "value": {"id": "prod-3", "quantity": 3}
}

Output:
{
  "cart": {
    "items": [
      {"id": "prod-1", "quantity": 2},
      {"id": "prod-2", "quantity": 1},
      {"id": "prod-3", "quantity": 3}
    ],
    "total": 35.98
  }
}

3. ARRAY_DELETE Example - Remove items from a list:
Input:
{
  "tags": ["important", "urgent", "review", "draft"]
}

Operation:
{
  "id": "removeTag",
  "operation": "EDIT_VALUE",
  "input": "getTags",
  "field": "tags",
  "sub_operation": "ARRAY_DELETE",
  "value": "draft"
}

Output:
{
  "tags": ["important", "urgent", "review"]
}

4. DICT_MERGE Example - Combine dictionaries:
Input:
{
  "user": {
    "name": "John Doe",
    "email": "john@example.com",
    "role": "user"
  }
}

Operation:
{
  "id": "updateUserProfile",
  "operation": "EDIT_VALUE",
  "input": "getUserProfile",
  "field": "user",
  "sub_operation": "DICT_MERGE",
  "value": {
    "role": "admin",
    "department": "Engineering"
  }
}

Output:
{
  "user": {
    "name": "John Doe",
    "email": "john@example.com",
    "role": "admin",
    "department": "Engineering"
  }
}

5. ARRAY_DICT_RENDER Example - Transform array structure:
Input:
{
  "products": [
    {"productId": "p-123", "productName": "Widget", "productType": "Hardware"},
    {"productId": "p-456", "productName": "Gadget", "productType": "Electronics"}
  ]
}

Operation:
{
  "id": "transformProducts",
  "operation": "EDIT_VALUE",
  "input": "getProducts",
  "field": "products",
  "sub_operation": "ARRAY_DICT_RENDER",
  "value": {
    "id": "{productId}",
    "name": "{productName}",
    "type": "default"
  },
  "transformations": {"id": "upper"}
}

Output:
{
  "products": [
    {"id": "P-123", "name": "Widget", "type": "default"},
    {"id": "P-456", "name": "Gadget", "type": "default"}
  ]
}

6. ASSIGN_SERIAL_NUMBERS Example - Add serial numbers to array elements:
Input:
{
  "tasks": [
    {"title": "Complete project", "assignee": "John"},
    {"title": "Review documentation", "assignee": "Sarah"},
    {"title": "Deploy to production", "assignee": "Mike"}
  ]
}

Operation:
{
  "id": "addTaskNumbers",
  "operation": "EDIT_VALUE",
  "input": "getTasks",
  "field": "tasks",
  "sub_operation": "ASSIGN_SERIAL_NUMBERS",
  "value": {
    "field_name": "task_id",
    "prefix": "TASK-"
  }
}

Output:
{
  "tasks": [
    {"title": "Complete project", "assignee": "John", "task_id": "TASK-1"},
    {"title": "Review documentation", "assignee": "Sarah", "task_id": "TASK-2"},
    {"title": "Deploy to production", "assignee": "Mike", "task_id": "TASK-3"}
  ]
}

7. ARRAY_DICT_TO_ARRAY Example - Convert an array of dictionaries to a flat array of values:
Input:
{
  "employees": [
    {"id": "E001", "name": "John Smith", "department": "Engineering"},
    {"id": "E002", "name": "Jane Doe", "department": "Marketing"},
    {"id": "E003", "name": "Robert Johnson", "department": "Finance"}
  ]
}

Operation:
{
  "id": "extractNames",
  "operation": "EDIT_VALUE",
  "input": "getEmployees",
  "field": "employees",
  "sub_operation": "ARRAY_DICT_TO_ARRAY",
  "value": {"key": "name"}
}

Output:
{
  "employees": ["John Smith", "Jane Doe", "Robert Johnson"]
}

8. ARRAY_TO_ARRAY_DICT Example - Convert a flat array of values into an array of dictionaries:
Input:
{
  "products": ["Widget", "Gadget", "Tool"]
}

Operation:
{
  "id": "convertToProductObjects",
  "operation": "EDIT_VALUE",
  "input": "getProductNames",
  "field": "products",
  "sub_operation": "ARRAY_TO_ARRAY_DICT",
  "value": {"key": "productName"}
}

Output:
{
  "products": [{"productName": "Widget"}, {"productName": "Gadget"}, {"productName": "Tool"}]
}

9. DICT_TO_ARRAY_DICT Example - Convert a dictionary into an array of key-value pairs:
Input:
{
  "key_value_dict": {
    "key1": "value1",
    "key2": "value2",
    "key3": "value3"
  }
}

Operation:
{
  "id": "convertDictToArray",
  "operation": "EDIT_VALUE",
  "input": "test_data",
  "field": "key_value_dict",
  "sub_operation": "DICT_TO_ARRAY_DICT",
  "value": {
    "key_name": "key",
    "value_name": "value"
  }
}

Output:
{
  "key_value_dict": [
    {"key": "key1", "value": "value1"},
    {"key": "key2", "value": "value2"},
    {"key": "key3", "value": "value3"}
  ]
}

7. STRING_TO_INTEGER - Convert a string to a number
Input: {"item": {"id": "123", "quantity_str": "5"}}
Operation:

{
    "id": "convertQuantity",
    "operation": "EDIT_VALUE",
    "input": "getItem",
    "field": "item.quantity",
    "sub_operation": "ASSIGN",
    "value": "{item.quantity_str}"
},
{
    "id": "convertQuantityToInt",
    "operation": "EDIT_VALUE",
    "input": "convertQuantity",
    "field": "item.quantity",
    "sub_operation": "STRING_TO_INTEGER"
}
Output: {"item": {"id": "123", "quantity_str": "5", "quantity": 5}}

8. STRING_TO_FLOAT - Convert a string to a float
9. STRING_TO_BOOLEAN - Convert a string to a boolean

10. CONVERT_TO_ARRAY - Split a string into a list
Input: {"user": {"tags": "admin,premium,active"}}
Operation:

{
    "id": "convertTagsToArray",
    "operation": "EDIT_VALUE",
    "input": "getUser",
    "field": "user.tags",
    "sub_operation": "CONVERT_TO_ARRAY",
    "value": {"separator": ","}
}
Output: {"user": {"tags": ["admin", "premium", "active"]}}

Notes:
- The input field must reference a valid ID from a previous workflow step.
- For nested fields, use dot notation (e.g., user.address.city).
- ARRAY_... operations require the target field to resolve to an array.
- DICT_... operations require the target field to resolve to an object.
- The field parameter is optional for some sub-operations like DICT_MERGE if the intent is to merge into the root input object.
- Parameter substitution is supported in the value field. Use type_hint to control quoting for numeric values if needed.