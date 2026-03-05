Define a PAGINATION operation step in a JSON workflow language that handles paginated API responses by making multiple REST calls to retrieve all pages of data.

Basic Structure:
{
  "id": string,
  "operation": "PAGINATION",
  "url": string,
  "method": string,
  "payload": object,
  "type": string (optional, default: "simple"),
  "records_per_page": string,
  "total_records": string,
  "max_pages": number (optional),
  "page_delay": number (optional),
  "payload_placeholders": object (optional),
  "metadata": object (optional),
  "query_params": object (optional, default: {}),
  "context": object (optional, default: {}),
  "type_hint": object (optional, default: {}),
  "retry_with_payload_step": string (optional),
  "max_payload_retries": integer (optional, default: 3)
}

Key Features:
- Automatically handles pagination by making multiple REST API calls
- Dynamically updates pagination parameters between requests using payload_placeholders
- Automatically manages pagination metadata and context
- Requires unique operation ID
- type defaults to "simple" if not provided which means next and previous metadata is used to update the payload.

Examples:

1. Client-Side Pagination (Reasoning Intent):
{
  "id": "getAllOrders",
  "operation": "PAGINATION",
  "url": "https://api.example.com/v1/orders",
  "method": "POST",
  "payload": {
    "page": {"direction": 1, "number": 1},
    "limit": 20,
    "filters": {
      "status": "active"
    }
  },
  "payload_placeholders": {
    "page.number": "metadata.next_page"
  },
  "max_pages": 5,
  "page_delay": 0.5,
  "records_per_page": "20",
  "total_records": "{getOrderCount.total}",
  "metadata": {"next": {"page": {
        "direction": 1}},
      "previous": {"page": {
        "direction": -1}}
    }
}

This will retrieve up to 5 pages of orders, updating the 'page.number' field in the payload
based on the 'metadata.next_page' value from each response.

2. Server-Side Pagination (Other Intents):
{
  "id": "getOrdersPage",
  "operation": "PAGINATION",
  "url": "https://api.example.com/v1/orders?page={security_params.current_page}",
  "method": "GET",
  "payload": {
    "limit": 50,
    "direction": 1
  },
  "records_per_page": "50",
  "total_records": "{getOrderCount.total}",
  "metadata": {"next": {
        "direction": 1},
      "previous": {
        "direction": -1}
    }
}

This retrieves a single page based on the current pagination context, which is
automatically managed across workflow executions.

3. Dynamic Token-Based Pagination:
{
  "id": "getTransactions",
  "operation": "PAGINATION",
  "url": "https://api.example.com/v1/transactions",
  "method": "POST",
  "payload": {
    "page_size": 100,
    "next_token": null
  },
  "payload_placeholders": {
    "next_token": "pagination.next_token"
  },
  "max_pages": 10,
  "records_per_page": "100",
  "total_records": "{getTransactionCount.count}"
}

Implementation Notes:
- The operation combines REST API calling with pagination logic
- For 'reasoning' intent, all page results are collected into an array
- For other intents, pagination context is fetched at the start and updated at the end
- payload_placeholders enables dynamic parameter updates between page requests
- metadata JSON object is used to store directional information for pagination.
- Supports `retry_with_payload_step` and `max_payload_retries` for LLM-based payload retry on API failure, same as the REST operation. On failure, the executor re-invokes the linked generation step (PAYLOAD, TEXT_TO_SQL, or TXT_TO_SOQL_QUERY) with error context, then retries the PAGINATION call.