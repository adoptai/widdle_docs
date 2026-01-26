Define a DATA_SOURCE_LOOKUP operation step in a JSON workflow language that searches organizational data sources using the assist bot.

Basic Structure:
{
  "id": string,
  "operation": "DATA_SOURCE_LOOKUP",
  "data_sources": list[string] (optional),
  "additional_instructions": string (optional),
  "max_results": integer (optional, default: 5),
  "score_threshold_abs": float (optional, default: 0.35),
  "score_threshold_rel": float (optional, default: 0.70)
}

Key Features:
- Uses user query automatically for searching
- Searches through organizational knowledge base and data sources
- Supports filtering by specific data sources/table names
- Returns markdown-formatted responses for better readability
- Uses assist bot's vector search capabilities with score-based filtering
- Includes chat history context for better responses

Parameters:
- data_sources: List of specific data source/table names to search in. If empty, searches all organizational data
- additional_instructions: Extra context for formatting the response
- max_results: Maximum number of results to return (default: 5)
- score_threshold_abs: Absolute score threshold for filtering results (default: 0.35)
- score_threshold_rel: Relative score threshold as percentage of best score (default: 0.70)

Examples:

1. Search All Data Sources:
{
  "id": "lookupCompanyInfo",
  "operation": "DATA_SOURCE_LOOKUP",
  "additional_instructions": "Focus on recent updates and provide actionable insights",
  "max_results": 5
}

2. Search Specific Data Sources:
{
  "id": "lookupProjectData",
  "operation": "DATA_SOURCE_LOOKUP",
  "data_sources": ["project_database", "task_management", "team_documents"],
  "max_results": 15
}

3. Filtered Search with Custom Instructions and Score Thresholds:
{
  "id": "lookupPolicyInfo",
  "operation": "DATA_SOURCE_LOOKUP",
  "data_sources": ["hr_policies", "compliance_docs"],
  "additional_instructions": "Summarize key policy changes and highlight any compliance requirements",
  "max_results": 8,
  "score_threshold_abs": 0.4,
  "score_threshold_rel": 0.75
}

Implementation Notes:
- The operation uses the user's query from the conversation context
- Results are filtered based on both absolute and relative score thresholds
- Chat history (last 5 messages) is included in markdown response generation
- Context format uses "title: {title}, content: {content}" for LLM processing
- Fallback to structured response if LLM generation fails
- Supports metadata-based data source filtering