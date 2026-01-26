Define a REST operation step in a JSON workflow language with the following specifications:

Basic Structure:
{
  "id": string,
  "operation": "REST",
  "additional_headers": object (optional, default: {}),
  "api_id": string (optional),
  "application": string (optional),
  "content_types": array (optional, default: []),
  "custom_acceptable_codes": array (optional, default: []),
  "files": array (optional, default: []),
  "headers": object (optional, default: {}),
  "method": string,
  "payload": object (optional),
  "query_params": object (optional),
  "response_format": string (optional),
  "timeout": number (optional, default: 60),
  "verify": boolean | string (optional, default: true),
  "url": string
}

Description:
- Supports GET, POST, PUT, PATCH, OPTIONS, POST_FORM requests
- Payload for POST, PUT, PATCH, POST_FORM requests. For GET requests, the payload will be converted to a 
  url-encoded query string.
- URL and payload parameter substitution using values from previous steps, or provided inputs.
- Dynamic path/query parameter replacement
- Id is a unique identifier for the operation. 
- Id is used to identify the result of the current operation, which is either a JSON object or an
  array.
- `api_id` and `application` parameters are used for browser-based API calls to identify the API
  and application context when making requests through the browser
- `verify` parameter controls SSL certificate verification. Defaults to `true` (verify certificates).
  Set to `false` to disable SSL verification, or provide a string path to a CA bundle file for
  custom certificate verification

Examples:
1. Simple GET request:
{
  "id": "getProjects",
  "operation": "REST",
  "url": "https://api.widdle.io/v1/projects",
  "method": "GET"
}

2. POST request with payload:
{
  "id": "createUser",
  "inputs": ["workflow_parameters"]
  "operation": "REST",
  "url": "https://api.widdle.io/v1/projects",
  "method": "POST",
  "payload": {
    "email": "{workflow_parameters.email}",
    "user_id": "{workflow_parameters.user_id}"
  },
  "query_params": {
    "limit": 10,
    "offset": 0
  }
}

3. Request with URL parameter substitution. Substitutes using field 'project_id' from the output
from output of the 'get_project_id' operation. If 'get_project_id' returns an array, this operation is invoked
on the 'project_id' field of each object in the array. The JSON outputs of each REST API is 
concatenated into an array.
{
  "id": "getProjectDetails",
  "operation": "REST",
  "url": "https://api.widdle.io/v1/projects/{get_project_id.project_id}/details",
  "method": "GET",
  "inputs": ["get_project_id"]
}

4. Browser-based API call with api_id and application:
{
  "id": "browserApiCall",
  "operation": "REST",
  "url": "https://api.example.com/v1/data",
  "method": "GET",
  "api_id": "example_api_v1",
  "application": "my_application"
}

5. POST request with SSL verification disabled:
{
  "id": "createUser",
  "operation": "REST",
  "url": "https://api.example.com/v1/users",
  "method": "POST",
  "verify": false,
  "payload": {
    "email": "user@example.com"
  }
}

6. GET request with custom CA bundle:
{
  "id": "getData",
  "operation": "REST",
  "url": "https://internal-api.example.com/v1/data",
  "method": "GET",
  "verify": "/path/to/ca-bundle.crt"
}