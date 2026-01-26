Define an EXECUTE_PLAN operation step in a JSON workflow language that executes a multi-step plan by delegating to a set of predefined actions.

Basic Structure:
{
  "id": string,
  "operation": "EXECUTE_PLAN",
  "plan": string,
  "actions": dict[string, string],
  "inputs": dict[string, string] (optional)
}

Key Features:
- Executes a comprehensive plan using available tools and actions
- The 'plan' field contains an LLM prompt that explains what the plan is, how to execute it, and what tools are available
- The 'actions' field maps action names to action IDs, providing a toolkit for plan execution
- The 'inputs' field maps resolvable outputs from previous steps to descriptive strings explaining their purpose in the plan
- Action names within this step must be unique to avoid conflicts
- Requires unique operation ID
- Used for complex multi-step operations that require orchestration of multiple actions

Examples:

1. Customer Onboarding Plan:
{
  "id": "executeOnboardingPlan",
  "operation": "EXECUTE_PLAN",
  "plan": "Execute a comprehensive customer onboarding plan. First, create the customer account using the account creation action. Then, set up their initial preferences using the preferences action. Finally, send a welcome email using the notification action. Ensure all steps complete successfully before proceeding to the next.",
  "actions": {
    "create_account": "action_123",
    "setup_preferences": "action_456", 
    "send_welcome_email": "action_789"
  },
  "inputs": {
    "getUserDetails": "Customer information needed for account creation",
    "getDefaultPreferences": "Default settings to apply during onboarding"
  }
}

2. Data Migration Plan:
{
  "id": "executeDataMigration",
  "operation": "EXECUTE_PLAN",
  "plan": "Perform a data migration from legacy system to new platform. Start by exporting data from the legacy system using the export action. Then validate the exported data using the validation action. If validation passes, import the data to the new system using the import action. Finally, run verification checks using the verify action to ensure data integrity.",
  "actions": {
    "export_legacy_data": "action_001",
    "validate_data": "action_002",
    "import_to_new_system": "action_003",
    "verify_migration": "action_004"
  },
  "inputs": {
    "getLegacySystemConfig": "Configuration details for the legacy system connection",
    "getTargetSystemConfig": "Target system settings and connection parameters",
    "getMigrationRules": "Business rules and data transformation mappings"
  }
}

Implementation Notes:
- The 'plan' field should be a clear, detailed description of the execution strategy
- The 'actions' field provides the tools available for executing the plan
- The 'inputs' field (optional) maps outputs from previous workflow steps to descriptive strings that explain their role in the plan
- Action names must be unique within the scope of this operation
- Input keys must reference valid IDs from previous workflow steps
- The operation coordinates the execution of multiple actions based on the plan logic
- Each action in the actions map should correspond to a valid action ID in the system
- The plan execution may involve conditional logic, error handling, and sequential or parallel action execution