Define an EXECUTE_LAMBDA operation step in a JSON workflow language with the following specifications:

Basic Structure:
{
  "id": string,
  "operation": "EXECUTE_LAMBDA",
  "lambda_name": string (mutually exclusive with lambda_id),
  "lambda_id": string (mutually exclusive with lambda_name),
  "input": object | string (optional, data passed to the lambda),
  "image": string (optional, overrides lambda's registered runtime_image),
  "timeout_seconds": number (optional, overrides lambda's registered timeout),
  "resource": {
    "cpu": string,
    "memory": string
  } (optional, overrides lambda's registered resource limits),
  "env": object (optional, custom environment variables merged with platform env),
  "image_auth": {
    "username": string,
    "password": string
  } (optional, for pulling private container images),
  "upload_files": array (optional, override lambda files from registry),
  "download_files": array (optional, download files from sandbox to S3)
}

upload_files entry structure (optional override):
{
  "path": string (destination path inside container, required),
  "content": string (inline file content, mutually exclusive with url),
  "url": string (URL to fetch and write, mutually exclusive with content)
}

Description:
- EXECUTE_LAMBDA runs a registered, reusable lambda function in an ephemeral first-party sandbox.
- Exactly one of "lambda_name" or "lambda_id" must be provided; specifying both is an error.
- Resolution flow:
  1. Look up the lambda in the platform registry by name or ID.
  2. Resolve config: WDL step values override lambda DB values, which override defaults.
     image: WDL "image" > lambda runtime_image > adopt-lambda-runtime
     timeout_seconds: WDL "timeout_seconds" > lambda timeout_seconds > 300
     resource: WDL "resource" > lambda cpu_limit/memory_limit > default
  3. Fetch the lambda's source files from S3 using the stored file_manifest.
  4. Create an ephemeral first-party sandbox using the resolved image.
     On-prem: only adopt-lambda-runtime is allowed. Cloud: any image is accepted.
  5. Upload registry files into /workspace/, then any WDL "upload_files" (overrides registry files).
  6. Write the "input" value as JSON to /workspace/_input.json (never passed as CLI args).
  7. Execute: python /workspace/{entry_point} /workspace/_input.json
  8. Capture stdout/stderr, parse output as JSON for structured output.
  9. Store execution record + S3 log. Destroy the sandbox.
- entry_point: Defaults to "script.py". Configurable per lambda registration. Must match
  pattern [a-zA-Z0-9_][a-zA-Z0-9_\-./]*.py and must not contain "..".
- input: Passed to the lambda as a JSON file (/workspace/_input.json), not as CLI arguments.
  Can be a JSON object literal or a {stepId} / {stepId.field} reference to a previous step output.
  String references like "{stepId}" or "{stepId.field}" are resolved from intermediate results.
  Unresolved references default to an empty object.
- Custom env (the "env" field): Optional dict of environment variables merged into the container.
  Platform env vars always win on conflict. Dangerous prefixes are blocked (ADOPT_, AWS_,
  SANDBOX_, SECRET_, TOKEN_, PYTHON, NODE_, RUBY, PERL, JAVA, etc.).
  Platform injects: ADOPT_ORG_ID, ADOPT_EXECUTION_TOKEN, ADOPT_PERMISSIONS.
- Network: Always deny-default with platform FQDN allowlist injected by the enforcer. Not
  configurable from WDL. This allows lambdas to call platform APIs (DB, vector store, documents)
  while blocking arbitrary outbound internet access.
- adopt_sdk is available inside the lambda runtime for structured logging and platform permission
  checking.
- On-prem deployments: Only the adopt-lambda-runtime image is allowed. All lambda executions
  on-prem are restricted to that image automatically.
- upload_files: Optional override. If provided, these files are uploaded instead of auto-fetching
  from the registry. Useful for testing or one-off invocations without formal registration.
- Step output fields:
  - execution_id: unique identifier for this execution
  - exit_code: exit code of the lambda process
  - duration_ms: wall-clock execution time in milliseconds
  - output: parsed JSON output, or raw string on parse failure
  - stdout: full, byte-exact standard output; `output` is parsed from it — full-string JSON first, then the last line (the old last-token split that corrupted JSON containing spaces is gone)
  - stderr: captured standard error from the lambda process
  - stdout_truncated: boolean; true means `output`/`stdout` are INCOMPLETE — do NOT trust them
  - sandbox_id: identifier of the ephemeral sandbox the lambda ran in
  - image: the resolved runtime image used for the execution
  - downloaded_files: list of presigned S3 URLs if download_files were requested
  - lambda_name: the lambda_name from the step (if provided)
  - lambda_id: the lambda_id from the step (if provided)
- Large output: return a small JSON status contract on stdout, not multi-MB data (it can truncate under load, surfacing as `stdout_truncated: true`). For bulk data, write it to a file/S3 and read it back with `PARSE_DOCUMENT(source_type:"s3") -> JQ_FILTER -> WRITE_TO_DB`.

When to use EXECUTE_LAMBDA vs SANDBOX:
- Use EXECUTE_LAMBDA when:
  - The code is registered, versioned, and reusable across workflows.
  - The lambda needs platform resource access (database, vector store, documents) via adopt_sdk.
  - First-party security model is required (scoped JWT, permission enforcement).
  - On-prem deployment compatibility is needed.
- Use SANDBOX when:
  - The code is ad-hoc, one-off, or user-defined at authoring time.
  - A custom Docker image is required (not available on-prem).
  - Full egress or specific FQDN allowlists are needed.
  - No platform resource access is required (pure isolation).

Examples:
1. Simple lambda execution by name:
{
  "id": "runAnalysis",
  "operation": "EXECUTE_LAMBDA",
  "lambda_name": "demand-forecast-etl"
}

2. Lambda with custom env vars:
{
  "id": "runWithEnv",
  "operation": "EXECUTE_LAMBDA",
  "lambda_name": "report-generator",
  "env": {
    "OUTPUT_FORMAT": "pdf",
    "LOCALE": "en_US"
  }
}

3. Lambda with input from a previous step:
{
  "id": "fetchData",
  "operation": "REST",
  "url": "https://api.example.com/records",
  "method": "GET"
},
{
  "id": "processRecords",
  "inputs": ["fetchData"],
  "operation": "EXECUTE_LAMBDA",
  "lambda_name": "record-processor",
  "input": "{fetchData}"
}

4. Pipeline combining EXECUTE_LAMBDA with other operations:
[
  {
    "id": "getConfig",
    "operation": "REST",
    "url": "https://api.example.com/config",
    "method": "GET"
  },
  {
    "id": "runEtl",
    "inputs": ["getConfig"],
    "operation": "EXECUTE_LAMBDA",
    "lambda_name": "etl-pipeline",
    "input": {
      "config": "{getConfig}",
      "run_date": "2026-04-07"
    }
  },
  {
    "id": "storeResult",
    "inputs": ["runEtl"],
    "operation": "WRITE_TO_DB",
    "table": "etl_results",
    "payload": {
      "execution_id": "{runEtl.execution_id}",
      "output": "{runEtl.output}"
    }
  }
]
