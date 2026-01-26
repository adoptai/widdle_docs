Define a M365_EMAIL_OPS operation step in a JSON workflow language that allows you to perform operations on Microsoft 365 emails.

Basic Structure:
{
  "id": string,
  "operation": "M365_EMAIL_OPS",
  "sub_operation": string,
  "mailbox_id_key_name": string,
  "tenant_id_key_name": string,
  "client_id_key_name": string,
  "client_secret_key_name": string,
  "table_name_key_name": string,
  "input_table_key_name": string,
  "prompt_addendum": string,
  "example_json": dict[string, string],
  "schema": list[dict[string, string]],
  "output_table_name_key_name": string,
  "epoch": integer,
  "ids": list[string] (optional)
}

Possible sub_operations:
- DOWNLOAD_M365_EMAIL: Download emails from the user's mailbox and index them in a vector store with LLM based summaries.
- SEARCH_M365_EMAIL: Search emails from the user's mailbox and return the results.
- SUMMARIZE_M365_EMAIL: Summarize emails from the user's mailbox and store the summaries in the database.


Generate these keys but leave the values empty strings. This is a features used only internal teams and not external users, so no need to provide any values.

