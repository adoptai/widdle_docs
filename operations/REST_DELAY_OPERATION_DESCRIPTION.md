Define a REST_DELAY operation step in a JSON workflow language that executes a REST call with a configurable initial delay and polls until the status field no longer matches the "in-progress" value.

Basic Structure:
{
  "id": string,
  "operation": "REST_DELAY",
  "url": string,
  "method": string,
  "payload": object (optional),
  "initial_delay": number (optional, default 10),
  "poll_interval": number (optional, default 5),
  "status_field": string (optional, default "data[0].status"),
  "running_status": string (optional, default "running"),
  "max_retries": integer (optional, default 20, max 20),
  ... (inherits all REST operation parameters)
}

Description:
- Executes a REST call after a configurable initial delay (default 10 seconds)
- Polls the endpoint at a configurable interval (default 5 seconds) until the status value changes
- The status value is read from a configurable JSON path in the response (default "data[0].status")
- Polling continues while the status equals a configurable "in-progress" value (default "running")
- Designed for asynchronous API operations that require time to complete
- Useful for long-running jobs, batch processing, or any API that returns a status field

Polling Configuration Fields:
- "initial_delay": seconds to wait before the first API call (default: 10)
- "poll_interval": seconds to wait between subsequent poll calls (default: 5)
- "status_field": dot/bracket path to the status value in the response JSON. Supports dot notation for nested keys and bracket notation for array indices. Examples: "status", "data.status", "data[0].status" (default: "data[0].status")
- "running_status": the status value that means "still in progress, keep polling". Polling stops when the resolved status is any value other than this (default: "running")
- "max_retries": maximum number of poll attempts after the initial call. When exceeded the last response is returned. Hard-capped at 20 to prevent runaway loops (default: 20)

Key Features:
- Configurable initial delay, poll interval, status path, and completion condition
- Backward-compatible defaults match the original hardcoded behaviour
- max_retries guard (default 20, hard-capped at 20) to prevent infinite polling
- Inherits all parameters from the standard REST operation
- Requires unique operation ID

Examples:

1. Poll with default settings (response wraps data in an array)
Input:
```json
{
  "reportId": "report-12345"
}
```

Operation:
```json
{
  "id": "waitForReport",
  "operation": "REST_DELAY",
  "method": "GET",
  "url": "/api/reports/{reportId}/status"
}
```

Output (after polling completes):
```json
{
  "data": [
    {
      "status": "completed",
      "reportUrl": "https://storage.example.com/reports/report-12345.pdf",
      "generatedAt": "2024-01-15T14:30:00Z"
    }
  ]
}
```

2. Poll an API that returns status at the root level
Input:
```json
{
  "jobId": "export-job-789"
}
```

Operation:
```json
{
  "id": "waitForExport",
  "operation": "REST_DELAY",
  "method": "GET",
  "url": "/api/exports/{jobId}",
  "status_field": "status",
  "running_status": "in_progress",
  "initial_delay": 5,
  "poll_interval": 3,
  "max_retries": 20
}
```

Output (after polling completes):
```json
{
  "status": "finished",
  "downloadUrl": "https://cdn.example.com/exports/data.csv",
  "rowCount": 15000
}
```

3. Poll a nested status path with a custom completion value
Operation:
```json
{
  "id": "pollDeployment",
  "operation": "REST_DELAY",
  "method": "GET",
  "url": "/api/deployments/{deployId}",
  "status_field": "result.meta.state",
  "running_status": "pending",
  "poll_interval": 10,
  "max_retries": 15
}
```

Implementation Notes:
- The "status_field" path is resolved against the top-level response JSON (resp[0] from _execute_rest_call)
- Supports dot notation ("data.status"), bracket notation ("data[0].status"), and plain keys ("status")
- Polling stops when the resolved status value is anything other than "running_status"
- If "max_retries" is exceeded, the operation returns the last response rather than looping forever. The value is always capped at 20 regardless of what is specified
- All REST operation parameters (method, url, payload, headers, etc.) are supported
- Consider API rate limits when choosing poll_interval
- The operation blocks the workflow until the status changes or max_retries is exceeded
- At default settings (10s initial + 20 x 5s polls), the maximum blocking time is ~110 seconds (~1.8 minutes)
