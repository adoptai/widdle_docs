Define an ASK_USER_FOR_INPUT operation step in a JSON workflow language. This is an interactive step that pauses the workflow to collect necessary information from the end-user by generating a form.

Basic Structure:
{
  "id": "string",
  "operation": "ASK_USER_FOR_INPUT",
  "prompt": "string (optional)",
  "fields": "string[] (optional)",
  "action_statement": "string (optional)"
}

Key Features:
- Interactive Data Collection: Pauses the workflow and waits for user input before proceeding.
- Dynamic Form Generation: Can generate a user-facing form in two ways:
-- LLM-Driven: Uses a natural language prompt to have an LLM construct the appropriate input fields.
-- Manually-Defined: Uses a predefined list of fields to request specific information.
- Context-Aware: Uses the action_statement and conversational history to provide context to the user about why the information is needed.
- Seamless Integration: The data submitted by the user becomes the output of this step and can be used by subsequent operations in the workflow.

Examples
1. LLM-Driven Form Generation (using prompt)
This example uses a prompt to ask an LLM to figure out the best way to ask the user for a reason for updating a ticket.

Operation:
{
  "id": "getUpdateReason",
  "operation": "ASK_USER_FOR_INPUT",
  "prompt": "Ask the user to provide a reason for their update. The reason should be a short text input."
}
Behavior: The workflow will pause. The user will be presented with a simple form containing a text field, likely labeled "Reason for update". Their entered text will become the output of the getUpdateReason step.

2. Manually-Defined Form (using fields)
This example explicitly requests a new user's name and email.

Operation:

{
  "id": "getNewUserDetails",
  "operation": "ASK_USER_FOR_INPUT",
  "action_statement": "To create a new user account",
  "fields": ["user_name", "user_email"]
}
Behavior: The workflow will pause. The user will see a form with two fields: "User Name" and "User Email". The output of the getNewUserDetails step will be an object like {"user_name": "Jane Doe", "user_email": "jane.doe@example.com"}.

Implementation Notes
- You must provide either a prompt or a fields list for the operation to function.
- The output of this step will be an object where keys are the form field names and values are the user-submitted data.
- Execution will resume after the user has submitted the form.