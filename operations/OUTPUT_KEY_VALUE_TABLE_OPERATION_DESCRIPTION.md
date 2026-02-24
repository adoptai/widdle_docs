Define an OUTPUT_KEY_VALUE_TABLE operation step in a JSON workflow language that generates a formatted key-value table from workflow data.

Basic Structure:
{
  "id": string,
  "operation": "OUTPUT_KEY_VALUE_TABLE",
  "data": object,
  "footer_message": string (optional),
  "formatting_rules": object (optional, default: {}),
  "header_message": string (optional),
  "input": string,
  "structured_context_key": string (optional)
}

Key Features:
- Creates a two-column table with field names and their corresponding values
- Takes output from a previous workflow step as input
- The 'data' object maps field paths to display names
- Automatically formats output as a GitHub-style markdown table
- Handles formatting of list values by joining them with commas
- Cleans HTML tags from string values

Example:
This example creates a key-value table from the first object in the "getProductDetails" result,
showing product name, price, availability and description with custom labels.
{
  "id": "showProductSummary",
  "operation": "OUTPUT_KEY_VALUE_TABLE",
  "input": "getProductDetails",
  "data": {
    "name": "Product Name",
    "price": "Price",
    "inventory.inStock": "Availability"
    "inventory description.inStock": "Description"
  }
}

Output Format:
The operation produces a GitHub-style markdown table with "Field" and "Value" headers:
| Field         | Value                      |
|---------------|----------------------------|
| Product Name  | Super Widget               |
| Price         | $19.99                     |
| Availability  | Yes                        |
| Description   | Premium quality widget     |

Notes:
- The 'input' field must reference a valid ID from a previous workflow step
- For nested fields, use dot notation (e.g., "inventory.inStock")
- Field paths can contain spaces (e.g., "inventory description.inStock")