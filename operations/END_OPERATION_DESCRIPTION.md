Define an END operation step in a JSON workflow language that terminates workflow execution and returns results collected up to that point.

Basic Structure:
{
  "id": string,
  "operation": "END",
  "key_field": string (optional),
  "value_field": string (optional)
}

Key Features:
- Immediately terminates the workflow execution
- Returns all intermediate results collected up to this point
- Used in conjunction with conditional logic to create early exit paths
- The id field is a unique operation ID

Example:
This example shows a workflow with conditional branching where END is used to terminate execution after the successful branch:

{
  "id": "checkProductAvailability",
  "operation": "CONDITION",
  "input": "inventoryData",
  "clauses": [
    {
      "field": "products",
      "operator": "!=",
      "value": []
    }
  ],
  "compound": "AND",
  "then": "displayProductDetails",
  "else": "showOutOfStockMessage"
},
{
  "id": "displayProductDetails",
  "operation": "EXTRACT_TRANSFORMED",
  "inputs": ["inventoryData"],
  "field": "/products/detail/{productId}"
},
{
  "id": "terminateWorkflow",
  "operation": "END"
},
{
  "id": "showOutOfStockMessage",
  "operation": "OUTPUT_TEXT",
  "format_string": "Sorry, the requested product is currently out of stock",
  "values": []
}

Implementation Notes:
- Any steps defined after the END operation are not executed
- This operation is particularly useful in workflows that have conditional branches
- When used with CONDITION operations, provides a way to exit workflows early
- Typically placed after a successful branch to prevent execution of the alternative branch
- Unlike most other operations, END does not produce any new intermediate results
- All accumulated intermediate results are preserved and returned when END is executed