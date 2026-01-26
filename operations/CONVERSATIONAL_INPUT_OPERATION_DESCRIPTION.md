Define a CONVERSATIONAL_INPUT operation step in a JSON workflow language that opens up an interactive conversation with the user to gather specific information.

Basic Structure:
{
  "id": string,
  "operation": "CONVERSATIONAL_INPUT",
  "prompt": string,
  "inputs": string[] (optional)
}

Key Features:
- Opens an interactive conversation with the user based on the provided prompt
- The prompt explains how to conduct the conversation, what to ask for, and when the goal is reached
- Can take multiple inputs from previous workflow steps to provide context for the conversation
- When complete, summarizes the conversation as a natural language paragraph for subsequent operations
- Requires unique operation ID
- The conversation continues until the goal specified in the prompt is achieved

Examples:

1. Gathering User Requirements:
{
  "id": "gatherRequirements",
  "operation": "CONVERSATIONAL_INPUT", 
  "prompt": "Have a conversation with the user to understand their software requirements. Ask about their preferred technology stack, project timeline, budget constraints, and key features they need. The conversation is complete when you have enough information to create a project specification.",
  "inputs": ["projectContext"]
}

2. Customer Support Information Gathering:
{
  "id": "supportInfo",
  "operation": "CONVERSATIONAL_INPUT",
  "prompt": "Conduct a support conversation to understand the user's technical issue. Ask about error messages, steps to reproduce the problem, their system configuration, and when the issue started. Continue until you have enough information to either solve the problem or escalate appropriately.",
  "inputs": ["userAccount", "previousTickets"]
}

3. Product Feedback Collection:
{
  "id": "collectFeedback", 
  "operation": "CONVERSATIONAL_INPUT",
  "prompt": "Engage the user in a conversation to collect detailed feedback about their experience with the product. Ask about what they liked, what could be improved, specific pain points, and suggestions for new features. The conversation should feel natural and encourage honest feedback.",
  "inputs": ["productUsageData"]
}

Implementation Notes:
- The 'prompt' field provides instructions for conducting the conversation
- The 'inputs' field is optional and references multiple previous workflow step results for context
- The operation returns a summary of the conversation as natural language text
- The conversation flow is adaptive based on user responses and the specified goals
- The system determines when enough information has been gathered based on the prompt requirements
- Output can be used by subsequent workflow operations for further processing