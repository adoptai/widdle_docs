Defines a FILTER operation in a JSON workflow language that filters array elements based on specified conditions.

Basic Structure:
{
  "id": string,
  "operation": "FILTER",
  "input": string,
  "compound": string ("AND" or "OR"),
  "clauses": [
    {
      "field": string,
      "operator": string,
      "value": any
    }
  ]
}

Key Features:
- Filters array of objects based on one or more conditions
- Supports compound conditions using AND/OR logic
- Takes output from previous workflow step as input
- Requires unique operation ID
- Produces new array containing only elements that match the conditions

Important:
- The input field must exactly equal to a previous output. It should not be a field of an output.
- If only a field is required, add an EXTRACT step to extract the field.

Examples:
1. Filter by Price and Category:
Input:
{
  "get_inventory": [
  {"product": "Phone", "category": "Electronics", "price": 699},
  {"product": "Laptop", "category": "Electronics", "price": 999},
  {"product": "Desk", "category": "Furniture", "price": 299},
  {"product": "Chair", "category": "Furniture", "price": 199},
  {"product": "Book", "category": "Books", "price": 20}
]
}
Operation:
{
  "id": "filterByPriceAndCategory",
  "operation": "FILTER",
  "input": "getInventory",
  "compound": "AND",
  "clauses": [
    {
      "field": "price",
      "operator": ">",
      "value": 15.00
    },
    {
      "field": "category",
      "operator": "==",
      "value": "Books"
    }
  ]
}

Output:
[
  {"product": "Book", "category": "Books", "price": 20}
]