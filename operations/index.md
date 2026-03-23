# WDL Operation Documentation Index

## Instructions for Autonomous Agents

You are navigating WDL (Widdle Definition Language) operation documentation.
Each `.md` file describes one operation for workflow definitions.

### Base URL

```
https://adoptai.github.io/widdle_docs/operations/
```

### Loading Strategy

1. **Analyze user request** - Identify needed capabilities
2. **Scan index below** - Find relevant operations
3. **Fetch documentation** - Append the filename from the table to the base URL

**Example:** To load the REST operation documentation:
```
https://adoptai.github.io/widdle_docs/operations/REST_OPERATION_DESCRIPTION.md
```

---

## All Operations

| Operation | File | Brief Description |
|-----------|------|-------------------|
| ADD_META_TO_CONTEXT | `ADD_META_TO_CONTEXT_OPERATION_DESCRIPTION.md` | adds metadata to the workflow context for later use |
| AGENTIC_SEARCH | `AGENTIC_SEARCH_OPERATION_DESCRIPTION.md` | performs vector similarity search across one or more vector tables and synthe... |
| ARITHMETIC | `ARITHMETIC_OPERATION_DESCRIPTION.md` | evaluates mathematical expressions using placeholder substitution from previo... |
| ASK_USER_FOR_INPUT | `ASK_USER_FOR_INPUT_OPERATION_DESCRIPTION.md` | pauses the workflow to collect necessary information from the end-user by gen... |
| BYO_SYSTEM_PROMPT | `BYO_SYSTEM_PROMPT_OPERATION_DESCRIPTION.md` | enables custom LLM interactions with a user-defined system prompt and model c... |
| CONDITION | `CONDITION_OPERATION_DESCRIPTION.md` | evaluates conditions and determines the next step based on the result |
| CONDITIONAL_ASSIGNMENT | `CONDITIONAL_ASSIGNMENT_OPERATION_DESCRIPTION.md` | selects between two previous operation results based on condition evaluation |
| CONVERSATIONAL_INPUT | `CONVERSATIONAL_INPUT_OPERATION_DESCRIPTION.md` | opens up an interactive conversation with the user to gather specific informa... |
| CREATE_MAP | `CREATE_MAP_OPERATION_DESCRIPTION.md` | transforms an array of objects into a key-value dictionary |
| CRYPTOGRAPHIC_OPERATION | `CRYPTOGRAPHIC_OPERATION_DESCRIPTION.md` | Supports RSA_OAEP encryption and decryption |
| DATA_SOURCE_LOOKUP | `DATA_SOURCE_LOOKUP_OPERATION_DESCRIPTION.md` | searches organizational data sources using the assist bot |
| DOWNLOAD_ENABLED | `DOWNLOAD_ENABLED_OPERATION_DESCRIPTION.md` | controls whether the workflow output can be downloaded by the user |
| EDIT_VALUE | `EDIT_VALUE_OPERATION_DESCRIPTION.md` | modifies JSON objects by adding, updating, or removing fields or array elements |
| EMBEDDER | `EMBEDDER_OPERATION_DESCRIPTION.md` | reads data from a source (table or S3 documents), chunks text, generates vect... |
| END | `END_OPERATION_DESCRIPTION.md` | terminates workflow execution and returns results collected up to that point |
| EXECUTE_PLAN | `EXECUTE_PLAN_OPERATION_DESCRIPTION.md` | executes a multi-step plan by delegating to a set of predefined actions |
| EXTRACT | `EXTRACT_OPERATION_DESCRIPTION.md` | extracts a single field from a JSON object |
| EXTRACT_AND_FLATTEN_UUID_MAP | `EXTRACT_AND_FLATTEN_UUID_MAP_OPERATION_DESCRIPTION.md` | transforms a UUID-keyed dictionary into a flat list of dictionaries |
| EXTRACT_STRUCTURED_CONTEXT_FROM_ARRAY | `EXTRACT_STRUCTURED_CONTEXT_FROM_ARRAY_OPERATION_DESCRIPTION.md` | extracts key-value pairs from array elements and stores them in a structured ... |
| EXTRACT_TRANSFORMED | `EXTRACT_TRANSFORMED_OPERATION_DESCRIPTION.md` | performs advanced data extraction and transformation using Python expressions... |
| FAN_OUT | `FAN_OUT_OPERATION_DESCRIPTION.md` | Use when you need to process each row with its own chain of operations (e.g. JQ_... |
| FETCH_META_FROM_CONTEXT | `FETCH_META_FROM_CONTEXT_OPERATION_DESCRIPTION.md` | retrieves previously stored metadata from the workflow context |
| FILTER | `FILTER_OPERATION_DESCRIPTION.md` | filters array elements based on specified conditions |
| FIRST_ELEMENT | `FIRST_ELEMENT_DESCRIPTION.md` | extracts the first element |
| FLATTEN | `FLATTEN_OPERATION_DESCRIPTION.md` | denormalizes nested JSON arrays |
| GROUP | `GROUP_BY_OPERATION_DESCRIPTION.md` | performs aggregation operations on grouped data |
| INTELLIGENT_FILTER | `INTELLIGENT_FILTER_OPERATION_DESCRIPTION.md` | uses AI-powered filtering to intelligently filter data based on natural langu... |
| INTELLIGENT_OUTPUT | `INTELLIGENT_OUTPUT_OPERATION_DESCRIPTION.md` | uses a Large Language Model (LLM) to intelligently format data into user-faci... |
| INTELLIGENT_OUTPUT_V2 | `INTELLIGENT_OUTPUT_V2_OPERATION_DESCRIPTION.md` | uses a Large Language Model (LLM) to intelligently format data into user-faci... |
| JQ_FILTER | `JQ_FILTER_OPERATION_DESCRIPTION.md` | to the output of previous steps. |
| JUMP | `JUMP_OPERATION_DESCRIPTION.md` | unconditionally transfers execution to another step in the workflow |
| M365_EMAIL_OPS | `M365_EMAIL_OPS_OPERATION_DESCRIPTION.md` | allows you to perform operations on Microsoft 365 emails |
| MERGE | `MERGE_OPERATION_DESCRIPTION.md` | combines two datasets based on specified key(s) |
| OUTLOOK | `OUTLOOK_OPERATION_DESCRIPTION.md` | LAST_N_MINUTES: Fetch emails from the last `lookback_minutes` (default 10). |
| OUTPUT_KEY_VALUE_TABLE | `OUTPUT_KEY_VALUE_TABLE_OPERATION_DESCRIPTION.md` | generates a formatted key-value table from workflow data |
| OUTPUT_TABLE | `OUTPUT_TABLE_DESCRIPTION.md` | formats data into a tabulated format with headers |
| OUTPUT_TEXT | `OUTPUT_TEXT_DESCRIPTION.md` | formats data into human-readable text using a template string |
| PAGINATION | `PAGINATION_OPERATION_DESCRIPTION.md` | handles paginated API responses by making multiple REST calls to retrieve all... |
| PAYLOAD | `PAYLOAD_OPERATION_DESCRIPTION.md` | uses a language model to generate structured JSON output conforming to a spec... |
| PAYLOAD_GENERATION | `PAYLOAD_GENERATION_OPERATION_DESCRIPTION.md` | AI-Powered Payload Creation: Leverages an LLM to create a JSON payload from natu... |
| POST_ACTION_URL | `POST_ACTION_URL_DESCRIPTION.md` | generates a URL where the user can view the results of their action, with opt... |
| PRE_ACTION_PAYLOAD_GENERATION | `PRE_ACTION_PAYLOAD_GENERATION_OPERATION_DESCRIPTION.md` | provides metadata for generating JSON payloads in subsequent REST steps |
| PROJECT | `PROJECTION_OPERATION_DESCRIPTION.md` | performs field selection on JSON arrays |
| PROMPT | `PROMPT_OPERATION_DESCRIPTION.md` | uses a language model to process data from a previous step based on given ins... |
| PROMPT_AND_TOOLS_AGENT | `PROMPT_AND_TOOLS_AGENT_OPERATION_DESCRIPTION.md` | creates an intelligent agent with a custom system prompt and access to specif... |
| READ_FROM_DB | `READ_FROM_DB_OPERATION_DESCRIPTION.md` | - TODO: Add when/why to use this operation |
| REASONING_METADATA | `REASONING_METADATA_OPERATION_DESCRIPTION.md` | configures reasoning behavior and step requirements for the workflow execution |
| REQUIRED_INPUTS | `REQUIRED_INPUTS_OPERATION_DESCRIPTION.md` | specifies input fields the user must provide before the workflow can execute |
| REST | `REST_OPERATION_DESCRIPTION.md` | Supports GET, POST, PUT, PATCH, OPTIONS, POST_FORM requests |
| REST_DELAY | `REST_DELAY_OPERATION_DESCRIPTION.md` | executes a REST call with an initial delay and polls until a non-running stat... |
| REST_LOOP | `REST_LOOP_OPERATION_DESCRIPTION.md` | executes multiple REST calls by iterating over a list of items with optional ... |
| S3_READ | `S3_READ_OPERATION_DESCRIPTION.md` | lists objects from an S3 folder, filters by file type and timestamp, batches ... |
| SORT | `SORT_OPERATION_DESCRIPTION.md` | orders array elements based on specified fields |
| STATEMENT | `STATEMENT_OPERATION_DESCRIPTION.md` | provides a human-readable description or title for the workflow action |
| SUGGESTIONS | `SUGGESTIONS_OPERATION_DESCRIPTION.md` | The id is always "suggestions". |
| TEXT_TO_SQL | `TEXT_TO_SQL_OPERATION_DESCRIPTION.md` | converts natural language queries into SQL queries using a language model |
| TOOL_EXECUTION | `TOOL_EXECUTION_OPERATION_DESCRIPTION.md` | executes an integration tool by looking up its specification from the databas... |
| TXT_TO_SOQL_QUERY | `TXT_TO_SOQL_QUERY_OPERATION_DESCRIPTION.md` | converts natural language user queries into valid Salesforce SOQL statements,... |
| UI_FORMAT_HINT | `UI_FORMAT_HINT.md` | provides hints for formatting the data of this workflow in a UI |
| VISUALISATION | `VISUALISATION_OPERATION_DESCRIPTION.md` | uses a Large Language Model (LLM) to generate Vega v6 chart specifications fr... |
| WRITE_TO_DB | `WRITE_TO_DB_OPERATION_DESCRIPTION.md` | - TODO: Add when/why to use this operation |

---

*Auto-generated on 2026-03-12 10:25:55*