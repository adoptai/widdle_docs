Define a SANDBOX operation step in a JSON workflow language with the following specifications:

Basic Structure:
{
  "id": string,
  "operation": "SANDBOX",
  "action": "init" | "exec" | "teardown",
  "image": string (required for init unless lambda_name/lambda_id provided),
  "lambda_name": string (optional, references a registered code snippet by name),
  "lambda_id": string (optional, references a registered code snippet by ID),
  "timeout_seconds": number (optional, overrides lambda's registered timeout),
  "env": object (optional, key-value pairs injected into container),
  "network": {
    "egress": "allow" | "deny" | array of FQDNs
  } (optional, init only, default: allow),
  "timeout_minutes": number (optional, default: 30, init only),
  "resource": {
    "cpu": string,
    "memory": string
  } (optional, init only, default: {"cpu": "1", "memory": "2Gi"}),
  "upload_files": array (optional),
  "download_files": array (optional),
  "get_endpoints": array of port numbers (optional),
  "image_auth": {
    "username": string,
    "password": string
  } (optional, init only, for pulling private container images),
  "command": string (required for exec),
  "new_sandbox": boolean (optional, init only; when true, tears down any existing session before creating a new one — defaults to reusing the current session)
}

upload_files entry structure:
{
  "path": string (destination path inside container, required),
  "content": string (inline file content, mutually exclusive with url),
  "url": string (URL to fetch and write, mutually exclusive with content)
}

download_files entry structure:
{
  "path": string (source path inside container)
}

Description:
- SANDBOX creates and manages an ephemeral container for arbitrary code execution.
- NOT available in on-premises deployments. Use EXECUTE_LAMBDA with the default runtime image instead.
- Three actions manage the container lifecycle: init, exec, teardown.
  - init: Creates and starts the container. Requires an "image" field. Sets network policy, resource
    limits, and injects env vars. The container persists across subsequent exec steps.
  - exec: Runs a command inside the existing container. Requires "command". Does not accept "image".
    Multiple exec steps share the same container session (filesystem state is preserved).
  - teardown: Destroys the container. Downloads any final files before destruction if "download_files"
    is specified. Always include a teardown step to release resources.
- The platform adopts a session model: init -> exec -> exec -> ... -> teardown all share the same
  container. The executor attaches to the running session across steps by step ID reference.
- "command" supports {stepId} and {stepId.field} references to inject values from previous step
  outputs. Resolved values are substituted directly and are not shell-escaped automatically.
  Apply appropriate quoting/escaping in the command when interpolating untrusted values.
- upload_files: Deploy files into the container before exec. Each entry specifies a container "path"
  and either inline "content" (string) or a "url" to fetch. Cannot specify both. Can be used on
  init or exec steps.
- download_files: Download files from the container to S3. Returns presigned S3 URLs in the step
  output under "downloaded_files". Can be used on exec or teardown steps.
- get_endpoints: Expose container ports and return proxied endpoint URLs. Each entry is a port number
  (integer). Returns a mapping of port -> proxied URL in the step output under "endpoints".
- Network policy (set on init only):
  - "egress": "allow" (default) — full outbound internet access.
  - "egress": "deny" — no outbound access.
  - "egress": ["api.example.com", "pypi.org"] — deny-default with explicit FQDN allowlist.
- Lambda snippet support: SANDBOX can reference registered code via "lambda_name" or "lambda_id"
  (mutually exclusive). When provided on init, the executor fetches the lambda's files from the
  registry and uploads them to /workspace/. The lambda's registered config (image, timeout,
  resource) becomes defaults that WDL-level values override. Inline "upload_files" override
  lambda files. Config resolution order: WDL step > Lambda registry > Default.
- adopt-lambda-runtime image is reserved for first-party use (EXECUTE_LAMBDA). Pure sandboxes
  cannot use that image.
- Dangerous env var prefixes are blocked (ADOPT_, AWS_, SANDBOX_, SECRET_, TOKEN_, etc.).
- Resource limits: "cpu" (e.g. "1", "2") and "memory" (e.g. "2Gi", "4Gi") in Kubernetes notation.
- Step output fields:
  - sandbox_id: identifier of the running container
  - exit_code: exit code of the executed command
  - stdout: captured standard output, reconstructed byte-exact from the SDK log chunks
  - stderr: captured standard error from the command
  - stdout_truncated: boolean; true when the SDK reported the stream was truncated — treat stdout as INCOMPLETE and do NOT parse it as authoritative
  - duration_ms: wall-clock execution time of the command in milliseconds
  - execution_id: identifier of this exec invocation
  - downloaded_files: list of presigned S3 URLs for downloaded files
  - endpoints: map of port number to proxied URL
- Large output: do NOT return multi-MB blobs or full CSV/JSON dumps via stdout. Even though stdout is byte-exact, it can still truncate under load (`stdout_truncated: true` plus a WARN). Keep stdout to a small status contract (e.g. `{"ok": true, "rows": 1234, "s3_url": "s3://..."}`) and write the real payload to a file, then `download_files` it (or write it to S3) and read it back with `PARSE_DOCUMENT(source_type:"s3") -> JQ_FILTER(.text|fromjson|...) -> WRITE_TO_DB`.

Examples:
1. Simple sandbox: init, run a script, teardown:
[
  {
    "id": "initSandbox",
    "operation": "SANDBOX",
    "action": "init",
    "image": "python:3.12-slim"
  },
  {
    "id": "runScript",
    "operation": "SANDBOX",
    "action": "exec",
    "command": "python3 -c \"print('hello world')\""
  },
  {
    "id": "teardownSandbox",
    "operation": "SANDBOX",
    "action": "teardown"
  }
]

2. Multi-step with file upload and download:
[
  {
    "id": "initSandbox",
    "operation": "SANDBOX",
    "action": "init",
    "image": "python:3.12-slim",
    "upload_files": [
      {
        "path": "/workspace/process.py",
        "content": "import json, sys\ndata = json.load(open(sys.argv[1]))\nprint(json.dumps({'count': len(data)}))"
      }
    ]
  },
  {
    "id": "runProcess",
    "operation": "SANDBOX",
    "action": "exec",
    "command": "python3 /workspace/process.py /workspace/input.json > /workspace/output.json"
  },
  {
    "id": "teardownSandbox",
    "operation": "SANDBOX",
    "action": "teardown",
    "download_files": [
      {"path": "/workspace/output.json"}
    ]
  }
]

3. Sandbox with network deny and FQDN allowlist:
[
  {
    "id": "initSandbox",
    "operation": "SANDBOX",
    "action": "init",
    "image": "python:3.12-slim",
    "network": {
      "egress": ["pypi.org", "files.pythonhosted.org"]
    }
  },
  {
    "id": "installAndRun",
    "operation": "SANDBOX",
    "action": "exec",
    "command": "pip install requests && python3 -c \"import requests; print(requests.__version__)\""
  },
  {
    "id": "teardownSandbox",
    "operation": "SANDBOX",
    "action": "teardown"
  }
]

4. Sandbox with {stepId} references in command:
[
  {
    "id": "getData",
    "operation": "REST",
    "url": "https://api.example.com/data",
    "method": "GET"
  },
  {
    "id": "initSandbox",
    "operation": "SANDBOX",
    "action": "init",
    "image": "python:3.12-slim"
  },
  {
    "id": "processData",
    "inputs": ["getData"],
    "operation": "SANDBOX",
    "action": "exec",
    "command": "python3 -c \"import json; data = json.loads({getData}); print(len(data))\""
  },
  {
    "id": "teardownSandbox",
    "operation": "SANDBOX",
    "action": "teardown"
  }
]
