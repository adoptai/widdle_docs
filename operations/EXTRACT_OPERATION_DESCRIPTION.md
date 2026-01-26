Defines an EXTRACT operation in a JSON workflow language that extracts a single field from a JSON object.

Basic Structure:
{
  "id": string,
  "operation": "EXTRACT",
  "input": string,
  "field": string
}

Key Features:
- Extracts a specified field from a JSON object. Does not work with arrays.
- Takes output from previous workflow step as input
- Produces a new object containing only the extracted field
- Requires unique operation ID.
- Format strings are in the Python str.format() style. For example: 
  "My name is {} and I'm {} years old".format(name, age). Use basic positional formatting,
  where the parenthesis are empty.

Example:
This example extracts the 'email' field from a user data object.
Input:
{
  "id": 123,
  "personalDetails": {
    "name": "John Doe",
    "email": "john.doe@example.com",
    "age": 30
  }
}

Operation:
{
  "id": "extractPersonalDetails",
  "operation": "EXTRACT",
  "input": "getUserData",
  "field": "personalDetails"
}

Output:
{
  "name": "John Doe",
  "email": "john.doe@example.com",
  "age": 30
}
This output can be accessed as extractPersonalDetails in subsequent steps. To print the above details,
use this:
{
  "id": "printPersonalDetails",
  "operation": "OUTPUT",
  "input": "extractPersonalDetails",
  "fields": ["name", "email", "age"],
  "format": "Name: {}, Email: {}, Age: {age}"
}