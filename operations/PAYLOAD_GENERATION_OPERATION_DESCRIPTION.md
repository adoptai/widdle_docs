Define a PAYLOAD_GENERATION operation step in a JSON workflow language. This operation uses a Large Language Model (LLM) to dynamically construct a single JSON object (a payload) based on a templated prompt.

Basic Structure:

{
  "id": "string",
  "operation": "PAYLOAD_GENERATION",
  "prompt": "string",
  "input_list": ["string", ...]
}

Key Features:
- AI-Powered Payload Creation: Leverages an LLM to create a JSON payload from natural language instructions and dynamic data.
- Templated Prompt: The prompt parameter acts as a template. It should contain placeholders in the format {variable_name}.
- Dynamic Data Injection: The input_list contains a list of JQ paths that extract data from previous workflow steps. This data is used to fill the placeholders in the prompt.
- Structured Output: The operation is constrained to always return a single, well-formed JSON object, which becomes the output of this step.

How it Works
This operation follows a specific process:
- It iterates through each string in the input_list. Each string is treated as a JQ path (e.g., "userDetails.name").
- It extracts the value from the workflow's intermediate results using that path.
- It makes the extracted value available to the prompt template. Crucially, the name of the placeholder in the prompt must exactly match the JQ path string from the input_list.
- The LLM receives the filled-in prompt and generates a JSON object according to the instructions.

Example
Previous Step Outputs in the Workflow:

// From step "getUser"
"getUser": {
  "id": "usr-123",
  "name": "Jane Doe",
  "access_level": "admin"
},

// From step "getSettings"
"getSettings": {
  "feature_flags": ["new_dashboard", "beta_reporting"]
}
Operation:
In this example, we want to create a payload to log an event. Notice how the placeholders in the prompt exactly match the JQ paths in the input_list.

{
  "id": "buildEventPayload",
  "operation": "PAYLOAD_GENERATION",
  "input_list": [
    "getUser.id", 
    "getUser.access_level",
    "getSettings.feature_flags"
  ],
  "prompt": "Create a JSON event payload for our analytics system. The event is 'user_viewed_page'. The user's ID is {getUser.id}, their access level is {getUser.access_level}, and the active feature flags are {getSettings.feature_flags}. The payload should include 'eventName', 'userId', and a 'metadata' object containing the other info."
}
Likely Output of the buildEventPayload step:

{
  "eventName": "user_viewed_page",
  "userId": "usr-123",
  "metadata": {
    "level": "admin",
    "flags": ["new_dashboard", "beta_reporting"]
  }
}
This generated payload is now available as buildEventPayload for use in a subsequent REST call or other operations.

Implementation Notes
Parameter Naming: The correct parameter names are prompt and input_list.
Placeholder Naming: This is a critical detail. The placeholder variables inside your prompt string (e.g., {getUser.id}) must be identical to the JQ path strings provided in the input_list.