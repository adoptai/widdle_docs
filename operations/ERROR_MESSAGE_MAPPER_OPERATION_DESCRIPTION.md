Defines an ERROR_MESSAGE_MAPPER operation in a JSON workflow language that registers a map of error codes to user-friendly messages, replacing raw error strings when a workflow step fails.

Basic Structure:
{
  "id": string,
  "operation": "ERROR_MESSAGE_MAPPER",
  "messages": {
    "<error_code>": string,
    "default": string
  }
}

Key Features:
- Registers a lookup table that maps HTTP/application error codes to human-readable messages
- Applied automatically whenever any subsequent step in the workflow encounters an error
- Falls back to the "default" key if the specific error code has no matching entry
- Falls back to the raw error string if neither the error code nor "default" is present in the map
- Produces no output of its own; its effect is to mutate the shared error message context for the workflow

Important:
- Place ERROR_MESSAGE_MAPPER early in the workflow, before any steps that may fail, so the map is registered before it is needed.
- The "default" key is a catch-all fallback and should always be included to ensure no raw error is surfaced to users.
- Error codes are matched as strings (e.g. "404", "500").

Examples:
1. Map common HTTP errors to friendly messages:
Operation:
{
  "id": "setErrorMessages",
  "operation": "ERROR_MESSAGE_MAPPER",
  "messages": {
    "404": "The requested record could not be found.",
    "403": "You do not have permission to perform this action.",
    "500": "An unexpected server error occurred. Please try again later.",
    "default": "Something went wrong. Please contact support if the issue persists."
  }
}

Effect:
If a later step fails with error code "404", the workflow surfaces "The requested record could not be found." instead of the raw error. If the error code is "429" (not in the map), the "default" message is used.

2. Minimal map with only a default:
Operation:
{
  "id": "setErrorMessages",
  "operation": "ERROR_MESSAGE_MAPPER",
  "messages": {
    "default": "Unable to complete the request. Please try again."
  }
}

Effect:
All errors, regardless of code, surface the single default message.
