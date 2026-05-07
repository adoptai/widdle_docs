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
- Supports scoping results to a specific document (by full filename or KB document title) via case-insensitive **exact match** on the vector metadata `source_label` or `title` field. File extensions are stripped from both the user-supplied value and the stored value before comparison, so `"Foo.pdf"` matches whether the indexed title is stored as `"Foo"` or `"Foo.pdf"`.
- Returns markdown-formatted responses for better readability
- Uses assist bot's vector search capabilities with score-based filtering
- Includes chat history context for better responses

Parameters:
- data_sources: List of full identifiers (a specific BYO filename or a specific KB document title) to scope the search. **Exact match only** — substrings, topic names, or file extensions alone do NOT match. A result is kept only if any provided value, after lowercasing and extension stripping, equals the result's `source_label` or `title`. Examples that work for a file uploaded as `Test_Rallyup_Campaign_Setup_Reference2.pdf`: `["Test_Rallyup_Campaign_Setup_Reference2"]` or `["Test_Rallyup_Campaign_Setup_Reference2.pdf"]`. Examples that do NOT work: `["Rallyup"]` (substring), `["Campaign_Setup"]` (substring), `[".pdf"]` (extension only). If `data_sources` is empty or omitted, the search runs across all organizational data.
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

2. Search Specific Data Sources (full filenames):
{
  "id": "lookupProjectData",
  "operation": "DATA_SOURCE_LOOKUP",
  "data_sources": ["RallyUp_Campaign_Setup_Reference", "iOS_18_All_New_Features_Sept_2024"],
  "max_results": 15
}

3. Filtered Search with Custom Instructions and Score Thresholds:
{
  "id": "lookupPolicyInfo",
  "operation": "DATA_SOURCE_LOOKUP",
  "data_sources": ["HR_Policy_Handbook_2026.pdf", "Compliance_Q1_Update"],
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