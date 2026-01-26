Define an ARITHMETIC operation step in a JSON workflow language that evaluates mathematical expressions using placeholder substitution from previous operation results.

Basic Structure:
{
  "id": string,
  "operation": "ARITHMETIC",
  "expression": string,
  "placeholders": object (optional, default: {})
}

Description:
- Performs arithmetic calculations on numeric values from previous workflow steps
- Uses placeholders to reference values from intermediate results, which are then substituted into the expression
- Evaluates the final mathematical expression and returns a numeric result
- Useful for computing derived values like totals, averages, percentages, or any numeric transformation

Key Features:
- Supports placeholder substitution from previous operation results using `{step_id.field}` syntax
- Placeholders can be strings (for substitution) or direct numeric values
- Evaluates standard mathematical expressions (+, -, *, /, etc.)
- Returns numeric output (integer or float)
- Requires unique operation ID

Examples:

1. Calculate Percentage
Input (from previous steps):
```json
{
  "counts": {
    "completed": 75,
    "total": 100
  }
}
```

Operation:
```json
{
  "id": "calculateCompletionRate",
  "operation": "ARITHMETIC",
  "placeholders": {
    "COMPLETED": "{counts.completed}",
    "TOTAL": "{counts.total}"
  },
  "expression": "(COMPLETED / TOTAL) * 100"
}
```

Output:
```json
75.0
```

2. Calculate Total Cost with Tax
Input (from previous steps):
```json
{
  "order": {
    "subtotal": 150.00,
    "taxRate": 0.08
  }
}
```

Operation:
```json
{
  "id": "calculateTotalWithTax",
  "operation": "ARITHMETIC",
  "placeholders": {
    "SUBTOTAL": "{order.subtotal}",
    "TAX_RATE": "{order.taxRate}"
  },
  "expression": "SUBTOTAL * (1 + TAX_RATE)"
}
```

Output:
```json
162.0
```

Implementation Notes:
- Placeholders are case-sensitive and must match exactly in the expression
- Placeholder values must resolve to numeric types (int or float)
- If a placeholder cannot be substituted (invalid reference), an error is returned
- The expression is evaluated after all placeholder substitutions are complete
- Complex expressions with parentheses and multiple operators are supported
- Division by zero will result in an error
