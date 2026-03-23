Define an AGENTIC_SEARCH operation step in a JSON workflow language that performs vector similarity search across one or more vector tables and synthesizes an answer using an LLM.

Basic Structure:
{
  "id": string,
  "operation": "AGENTIC_SEARCH",
  "input": string (reference to a previous step whose output data seeds the search context),
  "query": string (the natural-language question to search for across vector tables),
  "vector_tables": array of strings (optional; explicit list of vector table names to search),
  "vector_table_pattern": string (optional; SQL LIKE pattern to discover vector tables, e.g. "pipeline_abc_%_vec"),
  "instructions": string (optional; additional guidance for the LLM synthesis step),
  "reference_pipeline_id": string (optional; pipeline ID whose vector tables should also be searched — enables cross-pipeline RAG),
  "include_hitl_context": boolean (optional; when true, appends prior human corrections from HITL escalations to the search instructions for learning from past reviews; default false),
  "config": object (optional; overrides for embedding_type, top_k, score_threshold, llm_model, temperature, return_chunks — embedding_type defaults to the org's configured embedding when omitted)
}

Key Features:
- Searches across multiple vector tables simultaneously
- Tables can be specified as an explicit list or discovered via SQL LIKE pattern
- Retrieves the most similar chunks per table, merges and ranks by similarity score
- Filters results below a configurable score_threshold
- Synthesizes a final answer with citations using an LLM
- Citations include source_table, row_id, chunk_text, and relevance score
- query is used for vector search; instructions optionally guide the LLM synthesis
- Either vector_tables or vector_table_pattern (or both) must be provided; if neither is given and a prior PARSE_DOCUMENT step produced text, auto-embedding creates a vector table automatically
- reference_pipeline_id and include_hitl_context are TOP-LEVEL step keys, NOT inside config

Examples:

1. Search with Explicit Table List:
{
  "id": "searchFinancials",
  "operation": "AGENTIC_SEARCH",
  "query": "What are the key revenue trends for Q4?",
  "vector_tables": ["pipeline_abc_financials_vec"],
  "config": {
    "embedding_type": "Titan",
    "top_k": 10
  }
}

2. Search with Pattern and Instructions:
{
  "id": "searchAllDocs",
  "operation": "AGENTIC_SEARCH",
  "query": "Extract all employee W2 information",
  "vector_table_pattern": "pipeline_abc_%_vec",
  "instructions": "Summarize findings in a structured table format with names, wages, and tax withholdings",
  "config": {
    "embedding_type": "Titan",
    "top_k": 15,
    "score_threshold": 0.3,
    "return_chunks": true
  }
}

3. Search with LLM Override:
{
  "id": "deepSearch",
  "operation": "AGENTIC_SEARCH",
  "query": "What compliance risks exist in the submitted documents?",
  "vector_tables": ["pipeline_abc_docs_vec", "pipeline_abc_emails_vec"],
  "config": {
    "embedding_type": "Titan",
    "top_k": 20,
    "llm_model": "claude-sonnet",
    "temperature": 0.0
  }
}

4. Pipeline Search with Auto-Embed, Cross-Pipeline Tables, and HITL Context:
{
  "id": "agenticSearch",
  "operation": "AGENTIC_SEARCH",
  "input": "buildSearchStrategy",
  "query": "{{buildSearchStrategy.search_query}}",
  "reference_pipeline_id": "{{workflow_arguments.reference_pipeline_id}}",
  "include_hitl_context": true,
  "config": {
    "max_iterations": 3,
    "top_k": 15,
    "score_threshold": 0.2
  }
}

Implementation Notes:
- query is required; it is used for vector similarity search
- Either vector_tables (list) or vector_table_pattern (SQL LIKE pattern) may be provided; when both are omitted, the operation auto-embeds text from a prior PARSE_DOCUMENT step
- config.embedding_type should match the embedding used to create the vector tables; if omitted, defaults to the org's configured embedding
- config.top_k defaults to 10 chunks per table
- config.score_threshold defaults to 0.0 (no filtering)
- config.return_chunks defaults to true; when false, citations are omitted from output
- config.llm_model allows overriding the default LLM for synthesis
- The output includes answer, citations, tables_searched, and total_chunks_retrieved
- Errors during search or LLM synthesis are returned as error messages
