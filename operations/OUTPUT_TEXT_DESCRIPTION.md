Defines an OUTPUT operation in a JSON workflow language that formats data into human-readable text using a template string.
Basic Structure:
{
  "id": string,
  "operation": "OUTPUT_TEXT",
  "format_string": string,
  "values": array,
  "header_message": string (optional),
  "footer_message": string (optional),
  "formatting_rules": object (optional, default: {}),
  "raw": boolean (optional, default: false),
  "prompt": boolean (optional, default: false)
}
Key Features:

Takes a list of field references to extract values from previous workflow results.
Uses a format string template to combine the values into readable text
Supports both single values and structured data
Each field reference in "values" must point to existing data from previous steps
Format string must match the number of values being inserted
Uses Python-style string formatting with {} placeholders

Example:
This example formats user information into a readable message.
Previous Results:

saleInfo = {
"ItemName": "Book",
"price": 25.00,
},
userDetails = {
"userName": "John Smith",
"userAge": 25,
"userEmail": "john.smith@example.com"
},

Operation:
{
"id": "displayUserInfo",
"operation": "OUTPUT_TEXT",
"values": ["userDetails.userName", "userDetails.userEmail", "saleInfo.ItemName", "salesInfo.price"],
"format_string": "User {} with email ID {} purchased item {} for USD {}"
}
Output:
"User John Smith with email ID john.smith@example.com purchased item Book for USD 25.00"
Error Handling:

Returns an error if any referenced field is not found in previous results
Returns an error if format string doesn't match number of values
Returns an error if value extraction fails

Notes:

Field references must be valid JQ-style paths
Format string uses Python's str.format() syntax
Values are inserted in the order they appear in the "values" array
Field references within the "values" array should be plain field paths without braces. For example, use "formatReleaseMessage", not "{formatReleaseMessage}".
The operation produces a single string as output

Important:
- The 'input' field of any operation, if present, must equal the 'output' field of at least one preceding operation. 
It should not refer to the sub-field of an output field.