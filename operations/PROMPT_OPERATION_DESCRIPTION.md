Define a PROMPT operation step in a JSON workflow language that uses a language model to process data from a previous step based on given instructions.
This operation can be used as a standalone step or as part of a larger workflow.
When used as standalone (is_last_step: True), it apply the instructions to the input data (or conversation history) and returns the result in required output format thus not requiring OUTPUT_TEXT, OUTPUT_TABLE or OUTPUT_KEY_VALUE_TABLE operations.

Basic Structure:
{
  "id": string,
  "operation": "PROMPT",
  "input": string (optional),
  "inputs": array (optional),
  "instructions": string,
  "is_last_step": boolean (optional, default: true),
  "kwargs": object (optional, default: {}),
  "model_name": string (optional),
  "output_format": string (optional),
  "fields": array of strings or schema dicts (optional)
}

Key Features:
- Takes output from a previous workflow step as input
- Uses natural language instructions to direct the language model
- If input data is not provided, the operation will pick conversation history
- Processes the input data according to the specified instructions
- The 'is_last_step' field indicates if this is the final step in the workflow, defaults to True
- Returns the language model's response as output
- Requires unique operation ID

Examples:

1. Data Summarization:
{
  "id": "summarizeData",
  "operation": "PROMPT",
  "input": "getCustomerFeedback",
  "instructions": "Analyze this customer feedback data and provide a concise summary of the main pain points."
}

2. Data Analysis:
{
  "id": "analyzeFinancialData",
  "operation": "PROMPT",
  "input": "getQuarterlyReport",
  "instructions": "Identify the top 3 areas where costs increased compared to the previous quarter and suggest possible reasons."
}

3. Multiple Inputs with Custom Model:
{
  "id": "compareReports",
  "operation": "PROMPT",
  "inputs": ["currentQuarterData", "previousQuarterData"],
  "instructions": "Compare these two quarterly reports and highlight the key differences in revenue and expenses.",
  "model_name": "gpt-4o",
  "output_format": "Return as a JSON object with keys: revenue_change, expense_change, key_insights",
  "is_last_step": false
}

5. Structured Field Extraction — plain strings (intermediate step):
{
  "id": "extractDateRange",
  "operation": "PROMPT",
  "instructions": "Extract the date range mentioned by the user in the conversation.",
  "is_last_step": false,
  "fields": ["start_date", "end_date"]
}

6. Structured Field Extraction — dict with schema (enforces exact key names in nested output):
{
  "id": "extractFreightRates",
  "operation": "PROMPT",
  "input": "prepareEmailForPrompt",
  "instructions": "Extract all freight rate quotes from the email content. Return [] if no rate information is present.",
  "is_last_step": false,
  "fields": [{"extracted_rates": {"Carrier Name": "string or null", "Place of Loading": "string or null", "Place of Delivery": "string or null", "Base Rate": "float or null", "Currency": "string or null"}}]
}

4. Using kwargs for Model Parameters:
{
  "id": "creativeAnalysis",
  "operation": "PROMPT",
  "input": "marketingData",
  "instructions": "Generate creative campaign ideas based on this marketing data.",
  "kwargs": {
    "temperature": 0.8,
    "max_tokens": 1000
  }
}

Implementation Notes:
- Either 'input' (single step) or 'inputs' (multiple steps) can be used, but not both
- The 'input' field must reference a valid ID from a previous workflow step
- The input if specified, must be non-empty; the operation will return an error for empty inputs
- Instructions and output_format are limited to 200 words maximum for performance reasons
- The 'model_name' parameter allows specifying a different LLM model
- The 'kwargs' object allows passing additional parameters to the LLM (e.g., temperature, max_tokens)
- When 'is_last_step' is true and output is string/list, the result is formatted for display
- Timezone context is automatically passed to the LLM for date/time processing
- Errors in language model processing will be returned as error messages
- The 'fields' parameter enables structured key extraction: when provided and 'is_last_step' is false, the LLM extracts only the specified fields and returns a plain dictionary (e.g., {"start_date": "2025-01-01"}), suitable for use as input in subsequent steps
- Each entry in 'fields' can be either a plain string (e.g. "summary") or a dict whose key is the field name and value is a schema object defining exact key names and types for nested output (e.g. {"extracted_rates": {"Carrier Name": "string or null", "Base Rate": "float or null"}}). The dict form enforces that the LLM uses the exact key names in the schema rather than inventing its own based on source data terminology