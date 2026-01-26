Define an ADD_META_TO_CONTEXT operation step in a JSON workflow language that adds metadata to the workflow context for later use.

Basic Structure:
{
  "id": string,
  "operation": "ADD_META_TO_CONTEXT",
  "metadata": [
    {
      "name": string,
      "description": string,
      "value": any
    }
  ]
}

Key Features:
- Stores named metadata entities in the workflow context
- Takes values from previous workflow steps using parameter substitution
- Allows multiple metadata entries to be added in a single operation
- Each metadata entry requires a name, description, and value
- Values can be any valid JSON structure (objects, arrays, primitives)
- Stored metadata becomes available for use in subsequent workflow steps
- Requires unique operation ID

Examples:

1. Pagination Metadata Example:
{
  "id": "addPaginationMetadata",
  "operation": "ADD_META_TO_CONTEXT",
  "metadata": [
    {
      "name": "paging",
      "description": "Current page details which helps in pagination",
      "value": "{getContracts.meta}"
    },
    {
      "name": "sort",
      "description": "Current sort details which helps in sorting results",
      "value": "{updatedPayload.contracts_payload.sort}"
    }
  ]
}

Implementation Notes:
- The 'metadata' field must be a list of objects, each containing 'name', 'description', and 'value' properties
- The 'name' property defines the identifier by which the metadata can be accessed later
- The 'description' property provides documentation about the purpose of the metadata
- The 'value' property can contain direct values or references to values from previous workflow steps
- This operation is useful for storing configuration data, pagination information, user preferences, or any other metadata needed across multiple workflow steps