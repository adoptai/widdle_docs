Define an INTELLIGENT_FILTER operation step in a JSON workflow language that uses AI-powered filtering to intelligently filter data based on natural language criteria.

Basic Structure:
{
  "id": string,
  "operation": "INTELLIGENT_FILTER"
}

Description:
- Applies intelligent, AI-powered filtering to data from previous workflow steps
- Delegates to the IntelligentJQFilter component for smart data filtering
- Uses context from the execution environment including timezone information
- Useful when filter criteria are complex or need natural language interpretation

Key Features:
- AI-powered filtering using the IntelligentJQFilter component
- Accesses intermediate results from previous workflow operations
- Timezone-aware filtering for date/time related criteria
- Configuration-aware execution requiring a valid config instance
- Requires unique operation ID

Examples:

1. Filter Active Users from Last Week
Input (from previous steps):
```json
{
  "users": [
    {"name": "Alice", "lastActive": "2024-01-15T10:00:00Z", "status": "active"},
    {"name": "Bob", "lastActive": "2024-01-01T10:00:00Z", "status": "inactive"},
    {"name": "Charlie", "lastActive": "2024-01-14T15:30:00Z", "status": "active"}
  ]
}
```

Operation:
```json
{
  "id": "filterActiveUsers",
  "operation": "INTELLIGENT_FILTER"
}
```

Output:
```json
[
  {"name": "Alice", "lastActive": "2024-01-15T10:00:00Z", "status": "active"},
  {"name": "Charlie", "lastActive": "2024-01-14T15:30:00Z", "status": "active"}
]
```

2. Filter High-Priority Tasks
Input (from previous steps):
```json
{
  "tasks": [
    {"title": "Fix critical bug", "priority": "high", "due": "2024-01-16"},
    {"title": "Update docs", "priority": "low", "due": "2024-01-20"},
    {"title": "Security patch", "priority": "high", "due": "2024-01-15"}
  ]
}
```

Operation:
```json
{
  "id": "filterHighPriority",
  "operation": "INTELLIGENT_FILTER"
}
```

Output:
```json
[
  {"title": "Fix critical bug", "priority": "high", "due": "2024-01-16"},
  {"title": "Security patch", "priority": "high", "due": "2024-01-15"}
]
```

Implementation Notes:
- Requires a valid config_instance in the execution context
- Returns an error if no configuration is available
- The filtering logic is handled by the IntelligentJQFilter class
- Timezone context is automatically passed to handle date/time comparisons correctly
- Errors during filtering are caught and returned with full traceback
- This operation is typically used in conjunction with REST or other data-fetching operations
