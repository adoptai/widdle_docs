Define a FLATTEN operation step in a JSON workflow language that denormalizes nested JSON arrays.

Basic Structure:
{
  "id": string,
  "operation": "FLATTEN",
  "field_path": string,
  "input": string,
  "meta": array (optional)
}

Key Features:
- Transforms nested arrays into flat structures
- Combines parent object fields with each nested item
- Takes output from previous workflow step as input
- Uses dot notation for field paths
- Requires unique operation ID

Examples:

1. Order Details Example:
Input:
[{
  "order_id": "123",
  "customer": "John",
  "line_items": [
    {"product": "Widget", "quantity": 2, "price": 10},
    {"product": "Gadget", "quantity": 1, "price": 20}
  ]
}]

Operation:
{
  "id": "flattenOrderItems",
  "operation": "FLATTEN",
  "input": "getOrders",
  "field_path": ".line_items"
}

Output:
[
  {"order_id": "123", "customer": "John", "product": "Widget", "quantity": 2, "price": 10},
  {"order_id": "123", "customer": "John", "product": "Gadget", "quantity": 1, "price": 20}
]

2. Flatten with Meta Fields (pandas json_normalize):
Input:
[{
  "order_id": "456",
  "customer": "Jane",
  "items": [
    {"sku": "A001", "qty": 3},
    {"sku": "B002", "qty": 1}
  ]
}]

Operation:
{
  "id": "flattenWithMeta",
  "operation": "FLATTEN",
  "input": "getOrders",
  "field_path": "items",
  "meta": ["order_id", "customer"]
}

Output:
[
  {"sku": "A001", "qty": 3, "_order_id": "456", "_customer": "Jane"},
  {"sku": "B002", "qty": 1, "_order_id": "456", "_customer": "Jane"}
]

Implementation Notes:
- When 'meta' is not provided, uses legacy JQ-based flattening
- When 'meta' is provided, uses pandas json_normalize with meta_prefix='_'
- The 'field_path' uses dot notation for legacy mode (e.g., ".line_items")
- The 'field_path' uses path notation for meta mode (e.g., "items" or ["items", "subitems"])
- Meta fields are prefixed with '_' in the output to distinguish from nested fields