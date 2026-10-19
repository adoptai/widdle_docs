Define a COMPUTER_USE operation step in a JSON workflow language. COMPUTER_USE hands a goal to the Adopt extension's computer-use agent, which completes it in the user's own browser on their signed-in session by looking at the screen and clicking, typing, scrolling and navigating. Use it for web apps that have no usable API, for example downloading a statement from a bank portal.

Basic Structure:
{
  "id": string,
  "operation": "COMPUTER_USE",
  "goal": string (what the agent must achieve, in plain language; supports {placeholders} such as {workflow_arguments.period} or {stepId.field}),
  "start_url": string (optional; absolute http(s) URL the agent opens first; supports {placeholders}),
  "allowed_hosts": array of strings (optional; host names the agent may navigate to, "*.example.com" matches the domain and its subdomains; defaults to start_url's whole domain, e.g. "*.example.com" for www.example.com, or any host when start_url is absent; list them explicitly when the site uses other domains),
  "max_steps": number (optional; default 40, at most 80),
  "max_duration_seconds": number (optional; default 900, at most 1800)
}

Key Behavior:
- The step runs only in browser execution mode (org feature api_execution_mode) with the Adopt extension installed. Otherwise it fails with an error.
- The workflow pauses while the agent works in a new browser tab, placed in a tab group called "Adopt". If a sign-in page appears, the agent asks the user to sign in; it never types credentials.
- No one can answer the agent during the step, so if it needs to ask (for example before an irreversible action the goal did not explicitly ask for: submitting, paying, deleting), the step fails with its question.
- Requests from the agent's tab to the allowed hosts carry the header X-Adopt-Request-ID with the run's id, and the tab cannot load pages or frames outside allowed_hosts. Other requests a page makes, such as scripts, images and API calls, are not blocked.
- The step fails when the agent does not finish the goal (it stops, runs out of steps or time, or reports failure).

Output (the step's result, referenced as {stepId.field}):
{
  "status": "completed",
  "summary": string (the agent's answer or what it did),
  "steps": number,
  "final_url": string,
  "downloads": array of {"name": string, "bytes": number, "state": "complete" | "interrupted"}
}
Files the agent downloads are saved to the user's Downloads folder; only their names and sizes are returned. final_url has no query string or fragment.

The step shows nothing in the chat by itself: follow it with an output step (for example a PROMPT step with "input": "<stepId>") that tells the user the result.

Example, with the output step that follows it:
{
  "id": "downloadStatement",
  "operation": "COMPUTER_USE",
  "goal": "Download the account statement for the savings account for {workflow_arguments.period} as a PDF.",
  "start_url": "https://retail.onlinesbi.sbi/retail/login.htm",
  "allowed_hosts": ["*.onlinesbi.sbi"],
  "max_steps": 40,
  "max_duration_seconds": 600
},
{
  "id": "reportStatement",
  "operation": "PROMPT",
  "input": "downloadStatement",
  "instructions": "Tell the user in one or two sentences what was downloaded, naming each file.",
  "is_last_step": true
}
