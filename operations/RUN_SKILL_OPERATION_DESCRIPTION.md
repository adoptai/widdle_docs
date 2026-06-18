Defines a RUN_SKILL step that runs one or more Agent Skills via the agent harness as a sub-task within the pipeline.

Use this step when the pipeline should run skills by name. The skills are executed by an agent-harness turn (the LLM reads each skill's instructions and carries them out using its tools); when multiple skills are listed, the harness orchestrates between them in the appropriate order. The turn's text output is returned as this step's result.

Every name in `skills` MUST come from the "Available skills" list provided in the prompt context. Do not invent or hallucinate skill names.

All keys are at the top level of the step (not nested under "params").

Basic Structure:
{
  "id": string,
  "operation": "RUN_SKILL",
  "skills": array of strings (one or more skill names to run; must be from the available skills list),
  "user_message": string (optional; the instruction/prompt passed to the harness — describes the task the skills should accomplish),
  "model": string (optional; model override for the skill run; defaults to the platform default when omitted),
  "input": object (optional; maps values from upstream step results using {step_id.field} syntax),
  "output_step_id": string (optional),
  "notes": string
}

Key Features:
- Runs one or more skills by name in a single agent-harness turn; the harness loads the listed skills and orchestrates between them.
- user_message is the natural-language task handed to the harness; include any run-specific context or parameters here.
- model is an optional override; omit it to use the platform default.
- The step result is the harness turn's text output, usable by downstream steps.

Constraints:
- skills is required and MUST contain only names from the "Available skills" list.
- v1 supports skill pipelines as a single RUN_SKILL step. Do not mix RUN_SKILL with other operation types in the same pipeline.
