Define a TXT_TO_SOQL_QUERY operation step in a JSON workflow language that converts natural language user queries into valid Salesforce SOQL statements, executes them via a REST API call, and self-corrects on failure using a retry loop with LLM-driven error correction.

Basic Structure:
{
  "id": string,
  "operation": "TXT_TO_SOQL_QUERY",
  "object_schema": string,
  "additional_instructions": string (optional),
  "max_attempts": integer (optional, default: 3),
  "preferred_llm": string (optional),
  "sub_rest_operation": {
    "url": string,
    "method": string (optional, default: "GET"),
    "additional_headers": object (optional, default: {}),
    "field": string (optional, default: "records"),
    "query_param_key": string (optional, default: "q"),
    "timeout": integer (optional, default: 60)
  }
}

Description:
- Converts natural language user queries into valid SOQL (Salesforce Object Query Language) statements using an LLM
- Executes the generated SOQL query against a Salesforce REST API endpoint
- On failure (e.g., MALFORMED_QUERY, invalid field), feeds the HTTP error back to the LLM for self-correction
- Retries up to max_attempts times, accumulating conversation history so the LLM learns from its mistakes
- Uses a user-provided object schema to ensure the LLM only references valid fields for the target Salesforce org
- Designed to be portable across different Salesforce organisations with varying custom fields

Key Features:
- LLM-powered natural language to SOQL conversion with org-specific schema awareness
- Self-correcting retry loop: errors are fed back to the LLM as conversation context for correction
- Configurable REST endpoint via sub_rest_operation for flexibility across Salesforce instances and API versions
- Hardcoded SOQL rules covering syntax, field resolution, date formatting, query construction, and output formatting
- Field resolution strategy that maps natural language terms, abbreviations, and concepts to Field API Names
- Prevents field hallucination: the LLM is instructed to drop unresolvable conditions rather than invent field names
- Handles cross-field comparison limitations by generating broader queries for downstream INTELLIGENT_FILTER processing
- Supports optional additional_instructions for org-specific conventions or business logic hints
- Timezone-aware query generation
- Requires unique operation ID

Parameters:
- object_schema (required): The Salesforce object schema containing Field API Name, Field Label, and Data Type for the target object. This is org-specific and must be provided by the WDL author.
- additional_instructions (optional): Extra instructions appended to the system prompt for org-specific conventions, preferred field mappings, or business logic hints.
- max_attempts (optional, default: 3): Maximum number of LLM generation + REST execution attempts before returning an error.
- preferred_llm (optional): Override the default LLM. If not set, uses the intelligent_filtering_llm from config.
- sub_rest_operation (required): Configuration for the REST call that executes the generated SOQL query.
  - url (required): The Salesforce SOQL query API endpoint (e.g., https://instance.salesforce.com/services/data/v59.0/query)
  - method (optional, default: "GET"): HTTP method for the REST call
  - additional_headers (optional): Extra headers such as Authorization tokens, supports template substitution from previous steps
  - field (optional, default: "records"): The key in the JSON response from which to extract the result array
  - query_param_key (optional, default: "q"): The query parameter name for the SOQL string
  - timeout (optional, default: 60): Request timeout in seconds

Examples:

1. Basic Opportunity Query:
{
  "id": "textToSOQL",
  "operation": "TXT_TO_SOQL_QUERY",
  "object_schema": "Object: Opportunity\n\n| Field API Name | Field Label | Data Type |\n|---|---|---|\n| Id | Opportunity ID | Lookup() |\n| Name | Opportunity Name | Text(120) |\n| StageName | Stage | Picklist |\n| Amount | Total Contract Value | Currency(16, 2) |\n| CloseDate | Close Date | Date |\n| OwnerId | Opportunity Owner | Lookup(User) |",
  "sub_rest_operation": {
    "url": "https://instance.salesforce.com/services/data/v59.0/query",
    "additional_headers": {
      "Authorization": "Bearer {initiateQuery.access_token}"
    }
  }
}

2. With Additional Instructions and Custom Max Attempts:
{
  "id": "queryOpportunities",
  "operation": "TXT_TO_SOQL_QUERY",
  "object_schema": "Object: Opportunity\n\n| Field API Name | Field Label | Data Type |\n|---|---|---|\n| Id | Opportunity ID | Lookup() |\n| Engagement_Type__c | Engagement Type | Picklist |\n| Vertical_Industry__c | Vertical Industry | Picklist |\n| Amount | Total Contract Value | Currency(16, 2) |",
  "additional_instructions": "TCV refers to the Amount field. When filtering by engagement type, use exact values: Level1: T&M staffing, Level 3: Fixed Price Deliverable Based Projects, Level 4: Managed Services.",
  "max_attempts": 3,
  "preferred_llm": "claude_haiku",
  "sub_rest_operation": {
    "url": "https://instance.salesforce.com/services/data/v59.0/query",
    "method": "GET",
    "additional_headers": {
      "Authorization": "Bearer {initiateQuery.access_token}"
    },
    "field": "records",
    "query_param_key": "q",
    "timeout": 60
  }
}

Implementation Notes:
- Requires a valid config_instance in the execution context
- Returns an error if no configuration, object_schema, or sub_rest_operation url is available
- The user intent is extracted from intermediate_results via the USER_INTENT prerequisite (set in common.py)
- The retry loop maintains full conversation history across attempts so the LLM can learn from previous errors
- On each failure, an error feedback message containing the HTTP status code and Salesforce error response is appended to the conversation
- Empty LLM responses trigger a specific retry prompt asking for a valid SOQL string
- The rest_executor callable wraps the executor's _execute_rest_call method for authentication, browser-based calls, and response processing
- Records are extracted from the response using the configured field key (default: "records")
- SOQL responses are cleaned by stripping markdown code fences, trailing semicolons, and periods
- This operation is typically followed by an INTELLIGENT_FILTER step to handle complex filtering that SOQL cannot express (e.g., cross-field comparisons, correlations)
- Errors during execution are caught and returned with full traceback
- The operation is registered in WorkflowOperationType enum and has a corresponding Pydantic model (TxtToSoqlQueryBlock) for LLM-driven WDL generation
