Define an UI_FORMAT_HINT operation step in a JSON workflow language that provides hints for formatting the data of this workflow in a UI.
This should be defined towards the end of the workflow. 
This is used to provide hints to the UI about how to display the data.
This must be added to the workflow in a mandatory manner and the default for format_hint is "AI_DECIDES".

Basic Structure:
{
  "id": "ui_format_hint",
  "operation": "UI_FORMAT_HINT",
  "format_hint": string ("STACKED_VIEW" or "SIMPLE_LIST" or "TABLE" or "BULLETS_LIST" or "AI_DECIDES")
}