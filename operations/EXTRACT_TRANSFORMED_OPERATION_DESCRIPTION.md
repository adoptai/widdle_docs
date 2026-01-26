Define an EXTRACT_TRANSFORMED operation step in a JSON workflow language that performs advanced data extraction and transformation using Python expressions that resolve to JQ queries.

Basic Structure:
{
  "id": string,
  "operation": "EXTRACT_TRANSFORMED",
  "inputs": string[],
  "field": string
}

Key Features:
- Combines multiple input sources into a single context object
- Takes outputs from multiple previous workflow steps as inputs
- After parameter substitution, evaluates the 'field' as a Python expression to generate a JQ query
- Applies the resulting JQ query to transform the combined input data
- Produces a single output value based on the transformation
- The id field is a unique operation ID

Example with a Python dict that converts to JQ:
{
  "id": "createUserSummary",
  "operation": "EXTRACT_TRANSFORMED",
  "inputs": ["userData", "orderHistory"],
  "field": "{ 'name': '{.userData.name}', 'orderCount': '{.orderHistory}' }"
}

Example with dynamic parameter substitution:
{
  "id": "filterByCategory",
  "operation": "EXTRACT_TRANSFORMED",
  "inputs": ["Versions"],
  "field": "{Versions.latest_version} - 1"
}

Implementation Notes:
- The 'inputs' field is an array of IDs referencing valid previous workflow steps
- The 'field' parameter must be a valid Python expression that evaluates to either a string or dictionary after parameter substitution
- If field evaluates to a string, it's used directly as a JQ expression
- If field evaluates to a dictionary, it's converted to a JSON string before being processed
- Parameter substitution is applied to the field string before evaluation
- The Python eval() function is used to evaluate the field expression
- The resulting expression is passed to JQ for the actual data transformation
- The operation returns the first result of the JQ expression evaluation