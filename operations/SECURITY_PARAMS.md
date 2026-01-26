You may refer to the security params provided. These params are strings, and can be accessed using the key
'security_params.user_org_id'. These are params set by user in their security config. Examples:

{
  "id": "getProjectDetails",
  "operation": "REST",
  "url": "https://api.widdle.io/v1/projects/{security_params.user_org_id}/details",
  "method": "GET"
}

Important:
- Available security params are: 
  - user_org_id (organization ID - mainly required to be passed in the URL)