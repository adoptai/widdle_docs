Define a REST_LOOP operation step in a JSON workflow language that executes multiple REST calls by iterating over a list of items with optional delay between calls.

Basic Structure:
{
  "id": string,
  "operation": "REST_LOOP",
  "total": number (optional, default: 100),
  "multi_dict": array,
  "place_holder": object (optional, default: {}),
  "delay": number (optional, default: 0),
  ... (inherits REST operation parameters for the API call template)
}

Description:
- Iterates over a list of dictionaries (multi_dict) and executes a REST call for each item
- Supports placeholder substitution to inject item values into the REST call
- Optional delay between API calls to handle rate limiting
- Aggregates results from all REST calls into a single response
- Useful for batch operations like updating multiple records or fetching data for multiple IDs

Key Features:
- Batch REST call execution over a list of items
- Configurable delay between calls to respect API rate limits
- Placeholder substitution for dynamic URL/payload construction
- Parameter values can reference intermediate results using `{step_id.field}` syntax
- Returns aggregated results from all iterations
- Empty multi_dict returns success immediately
- Requires unique operation ID

Examples:

1. Fetch Details for Multiple Users
Input (from previous steps):
```json
{
  "userList": [
    {"userId": "user-001"},
    {"userId": "user-002"},
    {"userId": "user-003"}
  ]
}
```

Operation:
```json
{
  "id": "fetchUserDetails",
  "operation": "REST_LOOP",
  "total": 100,
  "multi_dict": "{userList}",
  "place_holder": {
    "USER_ID": "userId"
  },
  "delay": 0.5,
  "method": "GET",
  "url": "/api/users/{USER_ID}/details"
}
```

Output:
```json
{
  "success": true,
  "results": [
    {"id": "user-001", "name": "Alice", "email": "alice@example.com"},
    {"id": "user-002", "name": "Bob", "email": "bob@example.com"},
    {"id": "user-003", "name": "Charlie", "email": "charlie@example.com"}
  ]
}
```

2. Update Multiple Records with Rate Limiting
Input (from previous steps):
```json
{
  "recordsToUpdate": [
    {"recordId": "rec-1", "newStatus": "approved"},
    {"recordId": "rec-2", "newStatus": "approved"},
    {"recordId": "rec-3", "newStatus": "rejected"}
  ]
}
```

Operation:
```json
{
  "id": "batchUpdateRecords",
  "operation": "REST_LOOP",
  "total": 50,
  "multi_dict": "{recordsToUpdate}",
  "place_holder": {
    "RECORD_ID": "recordId",
    "STATUS": "newStatus"
  },
  "delay": 1.0,
  "method": "PATCH",
  "url": "/api/records/{RECORD_ID}",
  "payload": {
    "status": "{STATUS}"
  }
}
```

Output:
```json
{
  "success": true,
  "results": [
    {"recordId": "rec-1", "updated": true},
    {"recordId": "rec-2", "updated": true},
    {"recordId": "rec-3", "updated": true}
  ]
}
```

Implementation Notes:
- The `total` parameter specifies the maximum number of items expected (validation purposes)
- The `multi_dict` must be a list of dictionaries; it can be a string reference to intermediate results
- The `place_holder` object maps placeholder names to field names in each dictionary item
- The `delay` parameter is in seconds (supports decimals like 0.5 for 500ms)
- String parameters (total, multi_dict, place_holder, delay) support placeholder substitution
- If multi_dict is empty, the operation returns `{"success": true}` immediately
- Validation errors (non-list multi_dict, non-positive total) return appropriate error messages
- The operation aggregates results from all REST calls made during iteration
- Consider memory usage when processing large lists as all results are collected
