Define a PRE_ACTION_PAYLOAD_GENERATION operation step in a JSON workflow language that provides metadata for generating JSON payloads in subsequent REST steps.

Basic Structure:
{
  "id": string,
  "operation": "PRE_ACTION_PAYLOAD_GENERATION",
  "metadata": [
    {
      "type": "payload_generating",
      "metadata": {
        "example_payloads": list[object],
        "payload_generation_notes": string,
        "payload_name": string
      }
    }
  ]
}

Description:
- Defines how to generate payloads for REST operations that involve form submissions or complex data
- Provides example payloads and generation notes to guide the LLM in creating valid request bodies
- This is a metadata operation that doesn't execute an action itself but provides context for payload generation

Key Features:
- Contains example payloads that demonstrate the expected structure
- Includes notes explaining each field and validation requirements
- Payload name can be referenced in subsequent steps as workflow_parameters.payload_name
- Commonly used before REST operations that require POST/PUT/PATCH bodies
- Requires unique operation ID

Parameters:
- id: Unique identifier for this step
- operation: Must be "PRE_ACTION_PAYLOAD_GENERATION"
- metadata: Array of metadata objects containing:
  - type: Always "payload_generating"
  - metadata: Object with:
    - example_payloads: List of valid JSON payload examples
    - payload_generation_notes: String explaining field requirements
    - payload_name: Unique name to reference this payload

Examples:

1. Create User Payload Metadata
```json
{
  "id": "createUserPayloadMeta",
  "operation": "PRE_ACTION_PAYLOAD_GENERATION",
  "metadata": [
    {
      "type": "payload_generating",
      "metadata": {
        "example_payloads": [
          {
            "firstName": "John",
            "lastName": "Doe",
            "email": "john.doe@example.com",
            "department": "Engineering"
          }
        ],
        "payload_generation_notes": "firstName and lastName are required. email must be a valid email format. department should be one of: Engineering, Sales, Marketing, HR.",
        "payload_name": "create_user_payload"
      }
    }
  ]
}
```

2. Update Order Status Payload
```json
{
  "id": "updateOrderPayloadMeta",
  "operation": "PRE_ACTION_PAYLOAD_GENERATION",
  "metadata": [
    {
      "type": "payload_generating",
      "metadata": {
        "example_payloads": [
          {
            "orderId": "ORD-12345",
            "status": "shipped",
            "trackingNumber": "1Z999AA10123456784"
          }
        ],
        "payload_generation_notes": "orderId is required and must match existing order. status must be one of: pending, processing, shipped, delivered, cancelled. trackingNumber is required when status is 'shipped'.",
        "payload_name": "update_order_payload"
      }
    }
  ]
}
```
