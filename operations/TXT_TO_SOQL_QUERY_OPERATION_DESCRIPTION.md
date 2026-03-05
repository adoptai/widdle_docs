Define a TXT_TO_SOQL_QUERY operation step in a JSON workflow language that converts natural language user queries into valid Salesforce SOQL statements using an LLM. REST execution and retry-on-API-error are handled at the WiddleExecutor level via a separate REST step linked by retry_with_payload_step.

Basic Structure:
{
  "id": string,
  "operation": "TXT_TO_SOQL_QUERY",
  "object_schema": string,
  "additional_instructions": string (optional),
  "preferred_llm": string (optional)
}

Description:
- Converts natural language user queries into valid SOQL (Salesforce Object Query Language) statements using an LLM
- Returns the generated SOQL string; does NOT execute REST calls itself
- REST execution is handled by a separate REST/REST_DELAY/PAGINATION/REST_LOOP step in the workflow
- On API failure, the executor-level retry mechanism (retry_with_payload_step on the REST step) re-invokes this operation with error context so the LLM can self-correct
- Uses a user-provided object schema to ensure the LLM only references valid fields for the target Salesforce org
- Designed to be portable across different Salesforce organisations with varying custom fields

Key Features:
- LLM-powered natural language to SOQL conversion with org-specific schema awareness
- Error context support: when retried via the executor, api_error_context is appended to user intent so the LLM can correct its output
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
- preferred_llm (optional): Override the default LLM. If not set, uses the intelligent_filtering_llm from config.

Usage Pattern — Pair with a REST step for execution and automatic retry:
{
  "id": "gen_soql",
  "operation": "TXT_TO_SOQL_QUERY",
  "object_schema": "Object: Opportunity\n\n| Field API Name | Field Label | Data Type |\n|---|---|---|\n| Id | Opportunity ID | Lookup() |\n| Name | Opportunity Name | Text(120) |\n| StageName | Stage | Picklist |\n| Amount | Total Contract Value | Currency(16, 2) |"
}
followed by:
{
  "id": "exec_soql",
  "operation": "REST",
  "url": "https://instance.salesforce.com/services/data/v59.0/query",
  "method": "GET",
  "additional_headers": { "Authorization": "Bearer {initiateQuery.access_token}" },
  "query_params": { "q": "{gen_soql.query}" },
  "field": "records",
  "retry_with_payload_step": "gen_soql",
  "max_payload_retries": 3
}

Examples:

1. Basic Opportunity Query (generation only):
{
  "id": "textToSOQL",
  "operation": "TXT_TO_SOQL_QUERY",
  "object_schema": "Object: Opportunity\n\n| Field API Name | Field Label | Data Type |\n|---|---|---|\n| Id | Opportunity ID | Lookup() |\n| Name | Opportunity Name | Text(120) |\n| StageName | Stage | Picklist |\n| Amount | Total Contract Value | Currency(16, 2) |\n| CloseDate | Close Date | Date |\n| OwnerId | Opportunity Owner | Lookup(User) |"
}

2. With Additional Instructions and Custom LLM:
{
  "id": "queryOpportunities",
  "operation": "TXT_TO_SOQL_QUERY",
  "object_schema": "Object: Opportunity\n\n| Field API Name | Field Label | Data Type |\n|---|---|---|\n| Id | Opportunity ID | Lookup() |\n| Engagement_Type__c | Engagement Type | Picklist |\n| Vertical_Industry__c | Vertical Industry | Picklist |\n| Amount | Total Contract Value | Currency(16, 2) |",
  "additional_instructions": "TCV refers to the Amount field. When filtering by engagement type, use exact values: Level1: T&M staffing, Level 3: Fixed Price Deliverable Based Projects, Level 4: Managed Services.",
  "preferred_llm": "claude_haiku"
}

Implementation Notes:
- Requires a valid config_instance in the execution context
- Returns an error if no configuration or object_schema is available
- The user intent is extracted from intermediate_results via the USER_INTENT prerequisite (set in common.py)
- When invoked with api_error_context (during executor-level retry), the error details are appended to the user intent so the LLM can self-correct
- Empty LLM responses return an immediate error
- SOQL responses are cleaned by stripping markdown code fences, trailing semicolons, and periods
- This operation is typically followed by a REST step (with retry_with_payload_step pointing back) and then an INTELLIGENT_FILTER step to handle complex filtering that SOQL cannot express (e.g., cross-field comparisons, correlations)
- The operation is registered in WorkflowOperationType enum and has a corresponding Pydantic model (TxtToSoqlQueryBlock) for LLM-driven WDL generation
