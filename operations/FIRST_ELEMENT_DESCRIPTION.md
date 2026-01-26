Defines a First Element operation in a JSON workflow language that extracts the first element
from a JSON array.

Basic Structure:
{
  "id": string,
  "operation": "FIRST_ELEMENT",
  "input": string,
  "not_found_message": string
}

Input:
[
  {"product": "Phone", "category": "Electronics", "price": 699},
  {"product": "Laptop", "category": "Electronics", "price": 999},
  {"product": "Desk", "category": "Furniture", "price": 299},
  {"product": "Chair", "category": "Furniture", "price": 199}
]

Output:
{"product": "Phone", "category": "Electronics", "price": 699}

Description:
The contents of not_found_message are presented to the user if the input is empty. Set this to a useful
message, such as "Unable to find a user with the provided email Id", or "Could not find any product that
matches your request"