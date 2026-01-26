Define a REST_DELAY operation step in a JSON workflow language that executes a REST call with an initial delay and polls until a non-running status is detected.

Basic Structure:
{
  "id": string,
  "operation": "REST_DELAY",
  "url": string,
  "method": string,
  "payload": object (optional),
  ... (inherits all REST operation parameters)
}

Description:
- Executes a REST call after an initial 10-second delay
- Polls the endpoint every 5 seconds until the response status is no longer "running"
- Designed for asynchronous API operations that require time to complete
- Useful for long-running jobs, batch processing, or any API that returns a status field

Key Features:
- Initial 10-second delay before first API call
- Automatic polling every 5 seconds while status is "running"
- Returns the final response when status changes from "running"
- Inherits all parameters from the standard REST operation
- Requires unique operation ID

Examples:

1. Wait for Report Generation
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

2. Wait for Data Export Job
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
  "url": "/api/exports/{jobId}"
}
```

Output (after polling completes):
```json
{
  "data": [
    {
      "status": "finished",
      "downloadUrl": "https://cdn.example.com/exports/data.csv",
      "rowCount": 15000
    }
  ]
}
```

Implementation Notes:
- The operation expects the response to have a structure like `response.data[0].status`
- Polling continues indefinitely while `status === "running"` - ensure the API will eventually return a different status
- The initial delay is fixed at 10 seconds; the polling interval is fixed at 5 seconds
- All REST operation parameters (method, url, payload, headers, etc.) are supported
- Consider API rate limits when using this operation with frequently changing statuses
- The operation blocks the workflow until a non-running status is received
- Useful for APIs that process requests asynchronously and provide a status endpoint
