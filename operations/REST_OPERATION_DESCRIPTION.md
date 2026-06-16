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
  "url": string,
  "type_hint": object (optional, default: {}),
  "canonical_api_endpoint": string (optional),
  "retry_with_payload_step": string (optional),
  "max_payload_retries": integer (optional, default: 3),
  "capture_response_headers": boolean (optional, default: false),
  "via": string (optional),
  "tabby_profile_id": string (optional),
  "client_id": string (optional),
  "client_secret": string (optional)
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
- `retry_with_payload_step` links this REST step to a previous payload generation step (PAYLOAD, TEXT_TO_SQL, or TXT_TO_SOQL_QUERY). On API failure, the executor re-invokes that generation step with error context so the LLM can self-correct, then retries the REST call with the regenerated payload.
- `max_payload_retries` sets the maximum number of payload-regeneration-and-retry cycles (default: 3). Only used when `retry_with_payload_step` is set.
- `capture_response_headers` when set to `true`, stores the HTTP response headers in `intermediate_results` at `{step_id}__response_headers` with all keys lowercased. Use this when subsequent steps need to reference a response header (e.g. `{step_id__response_headers.mcp-session-id}`). Disabled by default to avoid unnecessary data in intermediate results.
- `via` routes the request through Tabby's authenticated browser session instead of a direct
  server-side HTTP call. Set `via: "tabby"` to bypass anti-bot / TLS-fingerprinting protection
  (Akamai, Cloudflare, PerimeterX): the request runs inside the live browser via Tabby's
  `/execute/fetch`, inheriting its TLS fingerprint and cookie jar. The response flows through the
  same normalization as a normal REST call; a binary response (e.g. application/pdf) is decoded and
  returned as a download reference. When `via` is absent the call executes server-side as before.
- `tabby_profile_id` (required when `via: "tabby"`) selects the Tabby profile / browser session to
  execute the request through.
  Tabby authentication is supplied by the platform at execution time: a per-user bearer token
  (obtained via the platform token-exchange) and the Tabby endpoint URL are passed into the
  executor and used automatically — you do not (and cannot) set them on the step. The Tabby
  endpoint is taken only from trusted sources (the platform-pushed auth or the `TABBY_API_URL`
  environment variable), never from the WDL, so an action cannot redirect the request or the
  bearer to another host. `client_id` / `client_secret` / `TABBY_CLIENT_ID` /
  `TABBY_CLIENT_SECRET` are only a self-host / local fallback used when no platform bearer is
  present.

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

7. REST call with LLM payload retry (paired with a PAYLOAD step):
{
  "id": "gen_payload",
  "operation": "PAYLOAD",
  "instructions": "Build a user creation payload from the input.",
  "json_schema": { "type": "object", "properties": { "email": { "type": "string" } }, "required": ["email"] },
  "input": "getUserInput"
}
{
  "id": "createUser",
  "operation": "REST",
  "url": "https://api.example.com/v1/users",
  "method": "POST",
  "payload": "{gen_payload}",
  "retry_with_payload_step": "gen_payload",
  "max_payload_retries": 3
}

8. MCP session initialization — capture response headers to pass session ID to subsequent steps:
{
  "id": "initialize",
  "operation": "REST",
  "url": "https://example.com/mcp",
  "method": "POST",
  "capture_response_headers": true,
  "payload": { "jsonrpc": "2.0", "id": 1, "method": "initialize", "params": {} }
}
{
  "id": "list_tools",
  "operation": "REST",
  "url": "https://example.com/mcp",
  "method": "POST",
  "additional_headers": { "Mcp-Session-Id": "{initialize__response_headers.mcp-session-id}" },
  "payload": { "jsonrpc": "2.0", "id": 2, "method": "tools/list" }
}

9. REST call with SOQL retry (paired with a TXT_TO_SOQL_QUERY step):
{
  "id": "gen_soql",
  "operation": "TXT_TO_SOQL_QUERY",
  "object_schema": "Object: Opportunity\n| Field API Name | Field Label | Data Type |\n| Id | Opportunity ID | Lookup() |\n| Name | Name | Text(120) |"
}
{
  "id": "querySalesforce",
  "operation": "REST",
  "url": "https://instance.salesforce.com/services/data/v59.0/query",
  "method": "GET",
  "query_params": { "q": "{gen_soql.query}" },
  "additional_headers": { "Authorization": "Bearer {auth.access_token}" },
  "retry_with_payload_step": "gen_soql",
  "max_payload_retries": 3
}