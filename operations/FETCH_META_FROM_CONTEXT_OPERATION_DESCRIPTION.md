Define a FETCH_META_FROM_CONTEXT operation step in a JSON workflow language that retrieves previously stored metadata from the workflow context.

Basic Structure:
{
  "id": string,
  "operation": "FETCH_META_FROM_CONTEXT",
  "metadata_names": string[]
}

Key Features:
- Retrieves named metadata entities that were previously stored in the workflow context
- Takes a list of metadata entity names to fetch
- Returns a dictionary with the requested metadata entities
- Only returns entities that exist in the context
- Requires unique operation ID
- Complements the ADD_META_TO_CONTEXT operation

Examples:

1. Fetch Pagination Metadata:
{
  "id": "fetchPaginationInfo",
  "operation": "FETCH_META_FROM_CONTEXT",
  "metadata_names": [
    "paging",
    "sort"
  ]
}

Implementation Notes:
- The 'metadata_names' field must be a list of strings representing entity names to retrieve
- We require this context ADD/FETCH when user wants to call same workflow multiple times but behaviour changes based on metdata values.
- Typical workflow pattern: add metadata with ADD_META_TO_CONTEXT towards end of workflow. Retrieve in the beginning of workflow with FETCH_META_FROM_CONTEXT. So that in every workflow run, you fetch metadata, use it(if available) and update it at the end.