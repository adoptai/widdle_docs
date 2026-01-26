Define a POST_ACTION_URL operation step in a JSON workflow language that generates a URL where the user can view the results of their action, with optional AI-powered URL construction.

Basic Structure:
{
  "id": string,
  "operation": "POST_ACTION_URL",
  "url_template": string,
  "input": string (optional),
  "prompt": string (optional),
  "examples": array (optional, default: []),
  "intelligent_filter_step_id": string (optional),
  "available_filters_step_id": string (optional)
}

Description:
- Generates a URL where users can view the results of their action in the source application
- Supports simple URL template substitution with values from previous steps
- Optionally uses LLM to intelligently construct URL parameters based on context
- Can incorporate filter information from INTELLIGENT_FILTER and REST operations

Key Features:
- URL template substitution using values from intermediate results
- AI-powered URL generation when 'prompt' is provided
- Integration with INTELLIGENT_FILTER for smart URL parameter construction
- Support for available filters from API responses
- User query context is automatically included when available
- Requires unique operation ID

Examples:

1. Simple URL Template Substitution:
Input from previous step "createUser":
{
  "id": "user-12345",
  "name": "John Doe"
}

Operation:
{
  "id": "viewUserUrl",
  "operation": "POST_ACTION_URL",
  "url_template": "https://app.example.com/users/{createUser.id}",
  "input": "createUser"
}

Output:
"https://app.example.com/users/user-12345"

2. AI-Powered URL Generation with Filters:
Operation:
{
  "id": "generateFilteredUrl",
  "operation": "POST_ACTION_URL",
  "url_template": "https://app.example.com/dashboard",
  "prompt": "Generate URL query parameters based on the user's filter request and available filter options",
  "examples": [
    {"input": "show active users", "output": "?status=active"},
    {"input": "filter by date range", "output": "?start_date=2024-01-01&end_date=2024-01-31"}
  ],
  "intelligent_filter_step_id": "applyFilters",
  "available_filters_step_id": "getFilterOptions"
}

Output:
"https://app.example.com/dashboard?status=active&department=engineering"

Implementation Notes:
- The 'url_template' is the base URL with optional placeholder substitution
- When 'prompt' is provided, an LLM is used to construct intelligent URL parameters
- The 'examples' array provides few-shot examples for the LLM
- 'intelligent_filter_step_id' references an INTELLIGENT_FILTER operation to get JQ filter context
- 'available_filters_step_id' references a REST operation that fetched available filter options
- The user's query is automatically retrieved from workflow_arguments when available
- Requires a heavy LLM to be configured when using the 'prompt' feature