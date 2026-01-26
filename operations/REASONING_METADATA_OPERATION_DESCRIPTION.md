Define a REASONING_METADATA operation step in a JSON workflow language that configures reasoning behavior and step requirements for the workflow execution.

Basic Structure:
{
  "id": string,
  "operation": "REASONING_METADATA",
  "is_reasoning_needed": boolean (optional, default: true),
  "skip": array (optional, default: []),
  "required": array (optional, default: []),
  "action_context": string (optional)
}

Description:
- Controls whether reasoning/explanation is needed for the workflow execution
- Specifies which steps can be skipped during execution
- Specifies which steps are required and must be executed
- Provides action context for better reasoning behavior
- Updates the S3 action manifest metadata for storage and tracking

Key Features:
- Toggle reasoning on/off with is_reasoning_needed flag
- Define arrays of step IDs that can be skipped
- Define arrays of step IDs that are required
- Optional action_context for additional execution context
- Updates intent type to JUST_ACTION_EXECUTION when reasoning is disabled
- Requires unique operation ID

Examples:

1. Configure Reasoning with Required and Skip Steps
Input (workflow context):
```json
{
  "workflow_steps": ["fetchData", "processData", "validateData", "outputResult", "sendNotification"]
}
```

Operation:
```json
{
  "id": "configureReasoning",
  "operation": "REASONING_METADATA",
  "is_reasoning_needed": true,
  "skip": ["sendNotification"],
  "required": ["fetchData", "processData", "outputResult"],
  "action_context": "Data processing workflow for quarterly reports"
}
```

Output:
```json
null
```

The workflow metadata is updated with the specified skip and required step arrays.

2. Disable Reasoning for Simple Actions
Input (workflow context):
```json
{
  "workflow_type": "simple_lookup"
}
```

Operation:
```json
{
  "id": "skipReasoning",
  "operation": "REASONING_METADATA",
  "is_reasoning_needed": false
}
```

Output:
```json
null
```

The workflow will execute without the reasoning phase, and intent is set to JUST_ACTION_EXECUTION.

Implementation Notes:
- The skip array must contain only string values (step IDs)
- The required array must contain only string values (step IDs)
- If is_reasoning_needed is false, the operation returns immediately without processing skip/required
- When reasoning is disabled, the intent type is automatically set to JUST_ACTION_EXECUTION
- Invalid types for skip or required arrays will return an error
- The action_context provides additional context that may be used by downstream reasoning processes
- This operation typically appears early in the workflow to configure execution behavior
