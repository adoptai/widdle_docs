Define a VISUALISATION operation step in a JSON workflow language that uses a Large Language Model (LLM) to generate Vega v6 chart specifications from data.

Basic Structure:
{
  "id": string,
  "operation": "VISUALISATION",
  "chart_hint": string (optional),
  "input": string,
  "preferred_llm": string (optional)
}

Key Features:
- Uses LLM to automatically determine the best chart type (bar, pie, or line)
- Generates valid Vega v6 JSON specification for chart rendering
- Aggregates large datasets for efficient visualization
- Returns chart spec in additional_data.visualisation_data
- Full data available for download

Parameters:
- input: ID of a previous workflow step that contains the data to be visualized (required)
- chart_hint: Optional hint for chart type ("bar", "pie", or "line"). LLM will still analyze data to confirm appropriateness.
- preferred_llm: Optional LLM model to use for generation

Chart Type Selection:
- BAR CHART: For comparing magnitudes across categories (e.g., vendor → spend, product → count)
- PIE CHART: For parts-of-whole with small number of categories (≤ 6-8 categories)
- LINE CHART: For time series or sequential data (e.g., date → revenue)

Output Structure:
The operation stores visualization data in FormattedMessage.additional_data.visualisation_data:
{
  "description": "Short 1-2 sentence description of what the chart shows (caption for user).",
  "visualisation_spec": { ... Vega v6 JSON ... },
  "chart_type": "bar" | "pie" | "line"
}

Examples:

1. Visualizing Vendor Spend Data:
Input from previous step "getVendorSpend":
[
  {"vendor": "Salesforce", "annual_spend": 50000},
  {"vendor": "AWS", "annual_spend": 120000},
  {"vendor": "Slack", "annual_spend": 15000}
]

Operation:
{
  "id": "visualizeSpend",
  "operation": "VISUALISATION",
  "input": "getVendorSpend"
}

Output:
A bar chart showing vendor names on X-axis and spend amounts on Y-axis.

2. Visualizing Monthly Trends:
Input from previous step "getMonthlyRevenue":
[
  {"month": "2024-01", "revenue": 100000},
  {"month": "2024-02", "revenue": 115000},
  {"month": "2024-03", "revenue": 98000}
]

Operation:
{
  "id": "visualizeTrend",
  "operation": "VISUALISATION",
  "input": "getMonthlyRevenue",
  "chart_hint": "line"
}

Output:
A line chart showing revenue trend over time.

Implementation Notes:
- The 'input' field must reference a valid ID from a previous workflow step
- Large datasets are automatically aggregated by the LLM for visualization
- The Vega v6 spec includes inline data (aggregated) for immediate rendering
- Full raw data is stored separately for download functionality
- If visualization generation fails, visualisation_data will be None