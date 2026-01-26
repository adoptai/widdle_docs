Define a TEXT_TO_SQL operation step in a JSON workflow language that converts natural language queries into SQL queries using a language model.

Basic Structure:
{
  "id": string,
  "operation": "TEXT_TO_SQL",
  "database_name": string,
  "table_name": string,
  "description": string,
  "schema": string,
  "sample_data": string (optional),
  "example_outputs": string (optional),
  "query_engine": string (optional, default: "apache_datafusion"),
  "preferred_llm": string (optional, default: "openai-new/gpt-5")
}

Key Features:
- Converts natural language user requests into executable SQL queries
- Uses LLM to understand the intent and generate appropriate SQL
- Supports multiple query engines (currently Apache DataFusion)
- Supports multiple LLM models (GPT, Claude, Google, Groq)
- Automatically uses fully qualified table names (database_name.table_name)
- Handles null values per-metric, not globally
- Generates clean SQL without markdown or JSON wrapping
- Requires unique operation ID

Parameters:
- database_name: The name of the database containing the target table (required)
- table_name: The name of the table to query (required)
- description: A description of the table and its purpose (required)
- schema: The schema definition of the table including column names, types, and constraints (required)
- sample_data: A sample data record from the table (required)
- example_outputs: Example outputs from the SQL query (optional)
- query_engine: The SQL query engine to use (optional, default: "apache_datafusion")
- preferred_llm: The LLM model to use for SQL generation (optional, default: "openai-new/gpt-5")

Query Engine Options:
- "apache_datafusion": Apache DataFusion query engine (default)

LLM Model Options:
- GPT models: "openai-new/gpt-5", "openai-new/gpt-4", "openai-new/gpt-4-turbo"
- Claude models: "anthropic/claude-4-5-sonnet", "anthropic/claude-3-sonnet"
- Google models: "google/gemini-pro", "google/gemini-2.5-flash"
- Groq models: "groq/llama3-8b", "groq/mixtral-8x7b"

Output:
The operation returns a JSON object with a "query" field containing the generated SQL query:
{
  "query": "SELECT column1, column2 FROM database_name.table_name WHERE condition"
}

Examples:

1. Basic SQL Query Generation: Find all users from the US region who have more than 10 sessions
{
  "id": "generateUserQuery",
  "operation": "TEXT_TO_SQL",
  "database_name": "analytics",
  "table_name": "user_metrics",
  "description": "Table containing user engagement metrics including user_id, session_count, and last_active_date",
  "schema": "user_id: string, session_count: integer, last_active_date: date, region: string",
  "sample_data": "user_id: '1234567890', session_count: 10, last_active_date: '2024-01-01', region: 'US'",
  "query_engine": "apache_datafusion",
  "preferred_llm": "openai-new/gpt-5"
}

2. Aggregation Query: Calculate the total sales amount per product category for the last 30 days
{
  "id": "generateAggregationQuery",
  "operation": "TEXT_TO_SQL",
  "database_name": "sales",
  "table_name": "transactions",
  "description": "Sales transactions table with transaction_id, amount, date, and product_category",
  "schema": "transaction_id: string, amount: decimal, date: date, product_category: string, customer_id: string",
  "preferred_llm": "anthropic/claude-4-5-sonnet"
}

3. Complex Analysis Query: Find the average latency and error rate for each model, but only include models with more than 100 requests
{
  "id": "generateAnalysisQuery",
  "operation": "TEXT_TO_SQL",
  "database_name": "metrics",
  "table_name": "gateway_model_metrics",
  "description": "Gateway model performance metrics including request_count, error_count, latency_ms, and timestamp",
  "schema": "request_id: string, request_count: integer, error_count: integer, latency_ms: decimal, timestamp: timestamp, model_name: string",
  "query_engine": "apache_datafusion"
}

Implementation Notes:
- The operation requires LLM_GATEWAY_API_KEY and LLM_GATEWAY_URL environment variables to be set
- The generated SQL query uses fully qualified table names (database_name.table_name)
- Null values are handled per-metric: records with null values in some metrics are still included if other metrics are valid
- The LLM is instructed to generate clean SQL without markdown formatting, JSON wrapping, or explanations
- Best practices are applied: CTEs for complex queries, explicit column names, descriptive aliases, early filtering
- The operation validates that database_name, table_name, and schema are provided
- Query engine and LLM model are optional with sensible defaults
- The generated SQL query is returned as a string in the "query" field of the result