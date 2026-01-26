Define a TOOL_EXECUTION operation step in a JSON workflow language that executes an integration tool by looking up its specification from the database and calling the external API endpoint with provided arguments.

Basic Structure:
{
  "id": string,
  "operation": "TOOL_EXECUTION",
  "input": object,
  "tool_name": string
}

Description:
- Executes integration tools that are registered in the system database
- Looks up the tool specification (URL, method, headers, authentication) by tool_name
- Calls the external API endpoint with the provided input arguments
- Supports various authentication methods (API keys, OAuth, etc.) configured per integration
- Returns the tool's response wrapped in an output object
- Useful for calling third-party APIs and services that are configured as integration tools

Key Features:
- Dynamic tool lookup from database by tool name
- Automatic authentication handling based on integration configuration
- Supports template placeholder substitution in URLs, headers, and query parameters
- Handles various HTTP methods (GET, POST, PUT, PATCH, DELETE)
- Returns structured output with tool response
- Requires unique operation ID

Examples:

1. Execute a Tool to Get User Information
Input:
```json
{
  "previous_step_result": {
    "user_id": "12345"
  }
}
```

Operation:
```json
{
  "id": "getUserInfo",
  "operation": "TOOL_EXECUTION",
  "tool_name": "get_user_details",
  "input": {
    "user_id": "{previous_step_result.user_id}",
    "include_profile": true
  }
}
```

Output:
```json
{
  "output": {
    "user_id": "12345",
    "name": "John Doe",
    "email": "john.doe@example.com",
    "profile": {
      "department": "Engineering",
      "role": "Senior Developer"
    }
  }
}
```

2. Execute a Tool with Complex Input Parameters
Input:
```json
{
  "workflow_parameters": {
    "project_id": "proj_789",
    "filters": {
      "status": "active",
      "date_range": "2024-01-01 to 2024-12-31"
    }
  }
}
```

Operation:
```json
{
  "id": "fetchProjectData",
  "operation": "TOOL_EXECUTION",
  "tool_name": "project_analytics",
  "input": {
    "project_id": "{workflow_parameters.project_id}",
    "filters": "{workflow_parameters.filters}",
    "format": "json"
  }
}
```

Output:
```json
{
  "output": {
    "project_id": "proj_789",
    "analytics": {
      "total_tasks": 150,
      "completed": 120,
      "in_progress": 25,
      "pending": 5
    },
    "generated_at": "2024-12-19T14:30:00Z"
  }
}
```

Implementation Notes:
- The `tool_name` must match a tool registered in the integration tools database
- The `input` parameter must be a dictionary/object (can be provided as JSON string which will be parsed)
- Tool specifications are retrieved from MySQL database including URL, HTTP method, headers, and authentication details
- Authentication secrets are automatically retrieved and applied based on the organization's integration configuration
- If the tool is not found in the database, the operation returns an error
- The tool's response is wrapped in `{"output": ...}` structure
- Supports parameter substitution in tool URLs and headers using `{param_name}` syntax
- Errors during tool execution are caught and returned as error messages
