You may refer to the workflow arguments provided. These arguments are strings, and can be accessed using the key
'workflow_arguments.name'. Examples:

{
  "id": "getProjectDetails",
  "operation": "REST",
  "url": "https://api.widdle.io/v1/projects/{workflow_arguments.project_id}/details",
  "method": "GET",
  "inputs": ["project_id"]
}

Important:
- Use only those workflow_arguments that have been provided. Do not make up your own.