Follow-up action suggestions. Displayed to the user after the current program has executed, to help them decide what to do next.

Basic Structure:
{
  "id": "suggestions",
  "operation": "SUGGESTIONS",
  "suggestions": "string" (must always be present, use empty string if no suggestions to provide),
  "actions": list[string] (must always be present, use empty list if no actions to suggest),
  "prompts": list[string] (must always be present, use empty list if no prompts to suggest),
  "limit": integer (must always be present, default -1)
}
- The id is always "suggestions".
- The operation is always "SUGGESTIONS".
- The suggestions are a string that is displayed to the user. Leave it blank, unless you have been provided with a list of suggestions when composing the program.
- The actions are a list of strings that represent action IDs the user can take. These are action UUIDs from the platform. Must always be generated as an empty list, unless you have been provided with a list of actions to suggest as follow ups when composing the program.
- The prompts are a list of strings that represent prompts the user can take. These are prompts that the user can select to continue the conversation. Must always be generated as an empty list, unless you have been provided with a list of prompts to suggest as follow ups when composing the program.
- The limit is an integer that represents the maximum number of suggestions that are auto generated beyong the actions and prompts to display to the user. Default is -1 (unlimited). Maximum is 5.