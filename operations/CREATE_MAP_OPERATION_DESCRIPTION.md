Define a CREATE_MAP operation step in a JSON workflow language that transforms an array of objects into a key-value dictionary.

Basic Structure:
{
  "id": string,
  "operation": "CREATE_MAP",
  "input": string,
  "key": string,
  "value": string
}

Key Features:
- Converts an array of objects into a dictionary (map)
- Uses specified fields from input array to create key-value pairs
- Takes output from a previous workflow step as input
- Requires unique operation ID
- Returns a dictionary with keys from the specified 'key' field and values from the specified 'value' field

Examples:

1. Create a Map of Product IDs to Prices:
Input:
[
  {"product_id": "P001", "name": "Laptop", "price": 999.99},
  {"product_id": "P002", "name": "Smartphone", "price": 599.99},
  {"product_id": "P003", "name": "Tablet", "price": 399.99}
]

Operation:
{
  "id": "createProductPriceMap",
  "operation": "CREATE_MAP",
  "input": "getProductList",
  "key": "product_id",
  "value": "price"
}

Output:
{
  "P001": 999.99,
  "P002": 599.99,
  "P003": 399.99
}

Implementation Notes:
- The 'input' field must reference a valid ID from a previous workflow step
- Input must be a non-empty array of objects
- 'key' and 'value' must be strings representing field names in the input objects
- If a key or value field is missing for any object, that object is skipped
- Duplicate keys will be overwritten by the last object with that key
- Returns an empty dictionary if no valid key-value pairs are found