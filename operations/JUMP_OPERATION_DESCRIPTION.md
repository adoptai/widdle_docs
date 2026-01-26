Define a JUMP operation step in a JSON workflow language that unconditionally transfers execution to another step in the workflow.

Basic Structure:
{
  "id": string,
  "operation": "JUMP",
  "target": string
}

Description:
- Unconditionally jumps to a specified step in the workflow
- Used to skip steps or create loops in workflow execution
- Unlike CONDITION, JUMP does not evaluate any conditions - it always jumps

Key Features:
- Immediately transfers control to the target step
- Target must be a valid step ID in the workflow
- Can be used after CONDITION to implement complex branching logic
- Useful for creating loops when combined with CONDITION
- Requires unique operation ID

Parameters:
- id: Unique identifier for this step
- operation: Must be "JUMP"
- target: The ID of the step to jump to

Examples:

1. Simple Jump to Another Step
Operation:
```json
{
  "id": "skipToOutput",
  "operation": "JUMP",
  "target": "displayResults"
}
```

2. Using JUMP with CONDITION for Loop-like Behavior
This example shows how JUMP can work with CONDITION to process items iteratively:
```json
[
  {
    "id": "checkMoreItems",
    "operation": "CONDITION",
    "input": "itemList",
    "clauses": [
      {
        "field": "remaining_count",
        "operator": ">",
        "value": 0
      }
    ],
    "compound": "AND",
    "then": "processNextItem",
    "else": "finishProcessing"
  },
  {
    "id": "processNextItem",
    "operation": "REST",
    "url": "/api/items/process"
  },
  {
    "id": "loopBack",
    "operation": "JUMP",
    "target": "checkMoreItems"
  },
  {
    "id": "finishProcessing",
    "operation": "OUTPUT_TEXT",
    "format_string": "All items processed",
    "values": []
  }
]
```
