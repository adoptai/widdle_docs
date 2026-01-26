Define a REQUIRED_INPUTS operation step in a JSON workflow language that specifies input fields the user must provide before the workflow can execute.

Basic Structure:
{
  "id": string,
  "operation": "REQUIRED_INPUTS",
  "required_inputs": list[string]
}

Description:
- Specifies which input fields must be collected from the user before workflow execution
- Generates a form with the specified fields for user input
- Collected values are available in subsequent steps via workflow_parameters

Key Features:
- Triggers a form generation to collect user inputs
- Each string in required_inputs becomes a form field
- Values are accessible in subsequent steps as workflow_parameters.<field_name>
- This is a metadata operation that defines data requirements
- Typically placed at the beginning of a workflow
- Requires unique operation ID

Parameters:
- id: Unique identifier for this step (commonly "required_inputs")
- operation: Must be "REQUIRED_INPUTS"
- required_inputs: Array of field names to collect from the user

Examples:

1. Basic User Information Collection
```json
{
  "id": "required_inputs",
  "operation": "REQUIRED_INPUTS",
  "required_inputs": ["user_id", "email"]
}
```
After this step executes, a form with 'user_id' and 'email' fields is displayed.
Subsequent steps can reference these values as:
- workflow_parameters.user_id
- workflow_parameters.email

2. Date Range and Status Filter
```json
{
  "id": "required_inputs",
  "operation": "REQUIRED_INPUTS",
  "required_inputs": ["start_date", "end_date", "status", "department"]
}
```
This generates a form for filtering data by date range, status, and department.

3. Using Required Inputs in a REST Call
```json
[
  {
    "id": "required_inputs",
    "operation": "REQUIRED_INPUTS",
    "required_inputs": ["employee_id", "review_period"]
  },
  {
    "id": "getEmployeeReviews",
    "operation": "REST",
    "url": "/api/employees/{{workflow_parameters.employee_id}}/reviews",
    "method": "GET",
    "query_params": {
      "period": "{{workflow_parameters.review_period}}"
    }
  }
]
```
