Define a DOWNLOAD_ENABLED operation step in a JSON workflow language that controls whether the workflow output can be downloaded by the user.

Basic Structure:
{
  "id": string,
  "operation": "DOWNLOAD_ENABLED",
  "fields_mapping": object (optional, default: {}),
  "value": boolean (optional, default: true)
}

Description:
- Sets metadata to control whether the workflow output is downloadable
- Useful for workflows that produce sensitive data or data that should only be viewed in the UI
- Affects the download functionality in the user interface

Key Features:
- Simple boolean toggle for download capability
- Defaults to true (downloads enabled) if not specified
- Updates workflow metadata immediately upon execution
- Requires unique operation ID

Examples:

1. Disable Downloads for Sensitive Report
Input:
```json
{
  "report": {
    "type": "salary_data",
    "employees": [...]
  }
}
```

Operation:
```json
{
  "id": "disableDownload",
  "operation": "DOWNLOAD_ENABLED",
  "value": false
}
```

Output:
```json
false
```

The workflow metadata is updated to prevent download functionality in the UI.

2. Explicitly Enable Downloads
Input:
```json
{
  "publicData": {
    "statistics": {...}
  }
}
```

Operation:
```json
{
  "id": "enableDownload",
  "operation": "DOWNLOAD_ENABLED",
  "value": true
}
```

Output:
```json
true
```

The workflow output will be available for download.

3. Enable Downloads with Field Mapping
Input:
```json
{
  "report": {
    "employee_data": [...],
    "summary": {...}
  }
}
```

Operation:
```json
{
  "id": "enableDownloadWithMapping",
  "operation": "DOWNLOAD_ENABLED",
  "value": true,
  "fields_mapping": {
    "employee_data": "employees",
    "summary": "report_summary"
  }
}
```

Output:
```json
true
```

The workflow output will be available for download with field names mapped according to the fields_mapping configuration.

Implementation Notes:
- The value parameter must be a boolean (true or false)
- If the value is not a boolean type, it defaults to true with a warning logged
- This operation updates the workflow's metadata.download.enabled property
- The `fields_mapping` parameter is an optional object that maps field names for download
- The fields_mapping is stored in metadata.download.fields_mapping
- The setting affects the UI's download button/functionality for the workflow output
- Typically placed early in the workflow or before output operations
