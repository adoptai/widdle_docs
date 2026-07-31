Define a PROMPT_AND_TOOLS_AGENT operation step in a JSON workflow language that creates an intelligent agent with a custom system prompt and access to specific ProjectA3 actions as tools.

Basic Structure:
{
  "id": string,
  "operation": "PROMPT_AND_TOOLS_AGENT",
  "simple_prompt_id": string (preferred, fetches prompt from prompt vault),
  "system_prompt": string (deprecated, use simple_prompt_id instead),
  "model_string": string (optional, default: "claude-4-6-sonnet"),
  "action_ids": list[string],
  "output_format": string (optional, one of: "message_only", "full_response", default: "message_only"),
  "inputs": list[string] (optional, keys from intermediate_results or workflow_arguments to include as additional context),
  "execution_timeout": number (optional, default: None),
  "max_iterations": number (optional, default: 15),
  "summarize_tool_context": boolean (optional, default: false),
  "skip_from_research": boolean (optional, default: false),
  "enable_form_tools": boolean (optional, default: false)
}

Key Features:
- Creates a LangChain-based agent with custom system prompt
- Provides the agent with specific ProjectA3 actions as callable tools
- Agent uses the user's query automatically from conversation context. When `inputs` is provided, resolved data from intermediate_results is appended as additional context to the query
- Supports multiple LLM models (GPT, Claude, Gemini, Groq)
- Automatically detects and uses LLM gateway when configured
- Agent can reason about which tools to use and when
- Handles tool execution results and generates coherent responses
- System prompts can be managed centrally in the prompt vault

Parameters:
- simple_prompt_id: ID of the prompt stored in the prompt vault (preferred approach for better prompt management)
- system_prompt: The system prompt that guides the agent's behavior (deprecated - use simple_prompt_id instead)
- model_string: LLM model identifier (e.g., "gpt-5", "claude-4-5-sonnet", "gemini-pro", "groq/llama3-8b")
- action_ids: List of ProjectA3 action IDs that the agent can use as tools
- output_format: How to format the response - "message_only" returns just the message, "full_response" returns message and finish status
- inputs: Optional list of keys from intermediate_results or workflow_arguments to resolve and append as additional context to the agent's query. Each key is resolved via jq from the workflow's intermediate results. When omitted, the agent uses only the user query from conversation context
- summarize_tool_context: When true, conversation history from previous turns is summarized via a fast LLM (Haiku) at the start of each new turn instead of loading raw tool messages into context. Raw data is always preserved in the DB. Useful for multi-turn agents that hit context/timeout limits after 3-4 turns
- skip_from_research: When true, prevents this agent's sub-action REST results from being forwarded into the parent workflow's deep research payload. Useful when an agent step gathers auxiliary data that should not be included in the final deep research analysis. Only relevant when the workflow runs in reasoning mode; has no effect otherwise. Defaults to false
- enable_form_tools: When true, enables agent-orchestrated structured forms. After the agent invokes sub-action tools, if a tool returns a valid `form_request` JSON spec (from a `STATIC_FORM` or `PAYLOAD` sub-action) and the agent status is `SUCCESS` or `CONTINUE`, the workflow pauses with `REQUESTING_USER_INPUT` and renders the form in chat via the existing FormRenderer. If the agent status is `FAILURE`, the form branch is skipped. On submit, the same agent step re-invokes with collected form values injected into context. Opt-in only; absent or false preserves existing text/CONTINUE behavior. Defaults to false

Model Options:
- GPT models: "gpt-5", "gpt-4", "gpt-4-turbo"
- Claude models: "claude-4-5-sonnet", "claude-4-6-sonnet"
- Gemini models: "gemini-pro", "gemini-3.6-flash"
- Groq models: "groq/llama3-8b", "groq/mixtral-8x7b"

Examples:

1. Customer Support Agent (Using Prompt Vault - Preferred):
{
  "id": "supportAgent",
  "operation": "PROMPT_AND_TOOLS_AGENT",
  "simple_prompt_id": "12345",
  "model_string": "claude-4-5-sonnet",
  "action_ids": ["get_user_details", "update_ticket_status", "search_knowledge_base"],
  "output_format": "message_only"
}

2. Data Analysis Agent (Legacy with Direct Prompt):
{
  "id": "dataAgent",
  "operation": "PROMPT_AND_TOOLS_AGENT",
  "system_prompt": "You are a data analysis expert. Help users analyze their data by fetching relevant information and generating insights. Present findings clearly with supporting details.",
  "model_string": "gpt-5",
  "action_ids": ["fetch_analytics_data", "generate_report", "export_to_csv"],
  "output_format": "full_response"
}

3. Task Management Agent (Using Prompt Vault):
{
  "id": "taskAgent",
  "operation": "PROMPT_AND_TOOLS_AGENT",
  "simple_prompt_id": "67890",
  "model_string": "gemini-pro",
  "action_ids": ["list_tasks", "create_task", "update_task", "delete_task"],
  "output_format": "message_only"
}

4. Webhook-Triggered Agent (With Inputs from workflow_arguments):
{
  "id": "webhookAgent",
  "operation": "PROMPT_AND_TOOLS_AGENT",
  "system_prompt": "You are triggered by a webhook. Your query includes an Additional context section containing workflow_arguments with an external_reference_id. Use it as the leadID when calling the enrichment tool.",
  "model_string": "claude-4-5-sonnet",
  "action_ids": ["enrich_lead"],
  "inputs": ["workflow_arguments"],
  "output_format": "message_only"
}

5. Agent With Multiple Inputs (Previous steps + workflow_arguments):
{
  "id": "multiInputAgent",
  "operation": "PROMPT_AND_TOOLS_AGENT",
  "simple_prompt_id": "11111",
  "model_string": "claude-4-5-sonnet",
  "action_ids": ["action_a", "action_b"],
  "inputs": ["workflow_arguments", "previousStepResult"]
}

6. Multi-Turn Agent With Context Summarization:
{
  "id": "uberAgent",
  "operation": "PROMPT_AND_TOOLS_AGENT",
  "simple_prompt_id": "99999",
  "model_string": "claude-4-5-sonnet",
  "action_ids": ["setup_environment", "process_data", "generate_output", "cleanup_session"],
  "summarize_tool_context": true,
  "output_format": "message_only"
}

Implementation Notes:
- PREFERRED: Use simple_prompt_id to reference prompts stored in the prompt vault for better prompt management and versioning
- LEGACY: system_prompt is still supported for backwards compatibility but should be migrated to simple_prompt_id
- The agent automatically uses the user query from the conversation context. When `inputs` is specified, the resolved data is appended as "Additional context" to the query
- Sub-action tools created from `action_ids` receive the parent's `conversation_history`, `user_query`, and `complete_user_query` so `PAYLOAD` steps inside form-spec sub-actions have LLM context (required for dynamic `form_request` generation)
- Security params, base URLs, and config instance are automatically provided from the workflow context
- The agent can make multiple tool calls in sequence to complete complex tasks
- Tool execution results are automatically formatted and presented to the agent
- The agent generates natural language responses based on tool outputs
- All tool calls are logged and tracked as part of workflow execution
- Supports both gateway-based and native LLM providers with automatic detection