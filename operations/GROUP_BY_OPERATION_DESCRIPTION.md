Define a GROUP operation step in a JSON workflow language that performs aggregation operations on grouped data.

Basic Structure:
{
  "id": string,
  "operation": "GROUP",
  "input": string,
  "fields": string[],
  "accumulator_op": string,
  "input_field": string,
  "output_field": string
}

Key Features:
- Groups array objects by specified field(s)
- Supports aggregation operations: COUNT, SUM, AVG, MIN, MAX
- Takes output from previous workflow step as input
- Requires unique operation ID
- Produces new array with grouped results

Examples:

1. Maximum Price by Company:
Input:
[
  {"price": 10, "company": "X"},
  {"price": 20, "company": "Y"},
  {"price": 30, "company": "X"}
]

Operation:
{
  "id": "maxPriceByCompany",
  "operation": "GROUP",
  "input": "getPrices",
  "fields": ["company"],
  "accumulator_op": "MAX",
  "input_field": "price",
  "output_field": "max_price"
}

Output:
[
  {"company": "X", "max_price": 30},
  {"company": "Y", "max_price": 20}
]