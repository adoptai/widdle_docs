Define a BYO_SYSTEM_PROMPT operation step in a JSON workflow language that enables custom LLM interactions with a user-defined system prompt and model configuration.

Basic Structure:
{
  "id": string,
  "operation": "BYO_SYSTEM_PROMPT",
  "system_prompt": string (optional, default: "You are a helpful assistant. Please answer the questions to the best of your ability."),
  "model_string": string (optional, default: "ByoModelStringType.openai_new_gpt_4o"),
  "key_name": string (optional, default: null),
  "api_key": string (optional, default: null),
  "params": object (optional, default: {}),
  "base_url": string (optional, default: null)
}

Description:
- Allows workflows to make custom LLM calls with a configurable system prompt
- Supports "Bring Your Own" (BYO) API keys for accessing different LLM providers
- Uses the conversation history from the workflow context for multi-turn interactions
- Enables specialized AI behaviors by customizing the system prompt

Key Features:
- Customizable system prompt for specialized AI assistant behaviors
- Support for multiple LLM models via model_string configuration
- Secure API key retrieval from secret store using key_name
- Alternatively accepts direct api_key for flexibility
- Custom base_url support for self-hosted or alternative LLM endpoints
- Additional model parameters via params object (temperature, max_tokens, etc.)
- Requires unique operation ID

Examples:

1. Custom Code Review Assistant
Input (conversation context):
```json
{
  "conversation_history": [
    {"role": "user", "content": "Review this Python function for potential issues: def add(a, b): return a + b"}
  ]
}
```

Operation:
```json
{
  "id": "codeReview",
  "operation": "BYO_SYSTEM_PROMPT",
  "system_prompt": "You are an expert Python code reviewer. Analyze code for bugs, security issues, and best practices. Provide concise, actionable feedback.",
  "model_string": "openai_new_gpt_4o",
  "key_name": "openai_api_key",
  "params": {
    "temperature": 0.3
  }
}
```

Output:
```json
"The function looks correct for basic addition. However, consider adding type hints for better code clarity: `def add(a: float, b: float) -> float:`. Also consider adding input validation if the function will be used with user input."
```

2. Custom Translation Service with Self-Hosted Model
Input (conversation context):
```json
{
  "conversation_history": [
    {"role": "user", "content": "Translate 'Hello, how are you?' to Spanish"}
  ]
}
```

Operation:
```json
{
  "id": "translator",
  "operation": "BYO_SYSTEM_PROMPT",
  "system_prompt": "You are a professional translator. Translate text accurately while preserving tone and context. Only output the translation, no explanations.",
  "base_url": "https://my-llm-server.internal/v1",
  "api_key": "local-api-key-123",
  "params": {
    "temperature": 0.1,
    "max_tokens": 500
  }
}
```

Output:
```json
"Hola, ¿cómo estás?"
```

Implementation Notes:
- The key_name parameter retrieves API keys securely from the organization's secret store
- If both key_name and api_key are provided, key_name takes precedence
- The conversation_history must be available in the execution context for multi-turn interactions
- The system_prompt must be a string; other types will cause an error
- The params object must be a dictionary; it's passed directly to the LLM provider
- Errors from the LLM provider are caught and returned with traceback information
