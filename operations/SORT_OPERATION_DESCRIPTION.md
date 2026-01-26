Define a SORT operation step in a JSON workflow language that orders array elements based on specified fields.

Basic Structure:
{
  "id": string,
  "operation": "SORT",
  "input": string,
  "fields": [
    {
      "name": string,
      "order": string ("ASC" or "DESC")
    }
  ]
}

Key Features:
- Sorts array of objects by one or more fields
- Supports ascending (ASC) and descending (DESC) order
- Takes output from previous workflow step as input
- Maintains original object structure
- Requires unique operation ID

Examples:
1. Multi-field Sort (Category and Price):
Input:
[
  {"product": "Phone", "category": "Electronics", "price": 699},
  {"product": "Laptop", "category": "Electronics", "price": 999},
  {"product": "Desk", "category": "Furniture", "price": 299},
  {"product": "Chair", "category": "Furniture", "price": 199}
]

Operation:
{
  "id": "sortByCategoryAndPrice",
  "operation": "SORT",
  "input": "getInventory",
  "fields": [
    {
      "name": "category",
      "order": "ASC"
    },
    {
      "name": "price",
      "order": "DESC"
    }
  ]
}

Output:
[
  {"product": "Laptop", "category": "Electronics", "price": 999},
  {"product": "Phone", "category": "Electronics", "price": 699},
  {"product": "Desk", "category": "Furniture", "price": 299},
  {"product": "Chair", "category": "Furniture", "price": 199}
]