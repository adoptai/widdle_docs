Description:
Defines a TABLE operation in a JSON workflow language that formats data into a tabulated format with headers.
Used when rendering multiple rows of output. You can control which fields are displayed using the headers field.
Basic Structure:
{
  "id": string,
  "operation": "OUTPUT_TABLE",
  "field_path": string (optional),
  "footer_message": string (optional),
  "formatting_rules": object (optional),
  "header_message": string (optional),
  "headers": object,
  "input": string,
  "limit": number (optional, default: 50),
  "meta": array (optional),
  "ordered_display_fields": array (optional)
}
Key Features:

Takes a list of records/arrays as input and formats them into a table
Requires headers to be specified for column names.
Uses GitHub-style markdown table formatting.
Takes output from previous workflow step as input.
Produces a string containing the formatted table.
Requires unique operation ID.
The ordered_display_fields field is used to control the order of the fields in the table. It is a list of table_heading strings (values of the headers field). Always specify this field for consistent user experience.
formatting_rules field is used to control the formatting of the fields in the table. It is a dictionary of field_name strings to formatting_rule strings.
The formatting_rule field is a dictionary with the following keys:
- type: string (date, map, currency, datetime, number, decimal_number, prefix, suffix)
- integer, and decimal_number have attribute decimal_places which is an integer (usually 2).
- prefix and suffix have attribute value which is a string.
- map is a dictionary of key-value pairs. When rendering, the value of the key is displayed.

Example of formatting_rule:

formatting_rule: {
  "currentTaskDueDate": {
    "type": "date",
  },
  "users": {
    "type": "number",
    "decimal_places": 2,
  },
  "cost": {
    "type": "currency",
    "decimal_places": 2,
  },
  "status": {
    "type": "map",
    "value": {
      "CANCELLED": "Cancelled",
      "COMPLETED": "Completed",
      "STARTED": "Open",
      "TERMINATED": "Denied",
    },
  },
}


Example:
This example creates a table from a list of user records.
Input:
"getUserList" = [
{name: "John Doe", age: 30, email: "john@example.com"},
{name: "Jane Smith",age: 25, email: "jane@example.com"},
{name: "Bob Wilson", age: 45, email: "bob@example.com"}
]
Operation:
{
"id": "userTable",
"operation": "OUTPUT_TABLE",
"input": "getUserList",
"headers": {"name": "Name", "age": "Age", "email": "Email"}
}
Output:
Name       Age Email
John Doe   30  john@example.com
Jane Smith 25  jane@example.com
Bob Wilson 45  bob@example.com

Notes:

The table formatting uses the tabulate library with GitHub-style markdown
Headers must match the structure of input data
Each row in the input must have the same number of columns as headers
Human readable names for the output.