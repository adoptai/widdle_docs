The status codes that are returned by the REST API are stored in the 'status_codes' field of the output and can be accessed using the key 'status_codes.getProjectDetails'.

Example Usage:
Previous REST Operation step ID: getProjectDetails
Usage:
{
  "id": "checkStatus",
  "operation": "CONDITION",
  "input": "status_codes",
  "compound": "AND",
  "clauses": [
    {
      "field": "getProjectDetails",
      "operator": "==",
      "value": 200
    }
  ],
  "then": "processLargeOrder",
  "else": "displayError"
}