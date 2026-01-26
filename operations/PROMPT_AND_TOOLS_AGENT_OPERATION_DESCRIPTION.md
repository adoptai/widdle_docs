Define a PROMPT_AND_TOOLS_AGENT operation step in a JSON workflow language that creates an intelligent agent with a custom system prompt and access to specific ProjectA3 actions as tools.

Basic Structure:
{
  "id": string,
  "operation": "PROMPT_AND_TOOLS_AGENT",
  "simple_prompt_id": string (preferred, fetches prompt from prompt vault),
  "system_prompt": string (deprecated, use simple_prompt_id instead),
  "model_string": string (optional, default: "claude-4-5-sonnet"),
  "action_ids": list[string],
  "output_format": string (optional, one of: "message_only", "full_response", default: "message_only")
}

Key Features:
- Creates a LangChain-based agent with custom system prompt
- Provides the agent with specific ProjectA3 actions as callable tools
- Agent uses the user's query automatically from conversation context
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

Model Options:
- GPT models: "gpt-5", "gpt-4", "gpt-4-turbo"
- Claude models: "claude-4-5-sonnet", "claude-3-sonnet"
- Gemini models: "gemini-pro", "gemini-2.5-flash"
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

Implementation Notes:
- PREFERRED: Use simple_prompt_id to reference prompts stored in the prompt vault for better prompt management and versioning
- LEGACY: system_prompt is still supported for backwards compatibility but should be migrated to simple_prompt_id
- The agent automatically uses the user query from the conversation context
- Security params, base URLs, and config instance are automatically provided from the workflow context
- The agent can make multiple tool calls in sequence to complete complex tasks
- Tool execution results are automatically formatted and presented to the agent
- The agent generates natural language responses based on tool outputs
- All tool calls are logged and tracked as part of workflow execution
- Supports both gateway-based and native LLM providers with automatic detection