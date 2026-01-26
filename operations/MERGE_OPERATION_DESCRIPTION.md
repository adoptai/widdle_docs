Define a MERGE operation step in a JSON workflow language that combines two datasets based on specified key(s).

Basic Structure:
{
  "id": string,
  "operation": "MERGE",
  "left_input": string,
  "right_input": string,
  "left_on": string[],
  "right_on": string[],
  "merge_type": string ("inner" | "left" | "right" | "outer"),
  "left_suffix": string (optional),
  "right_suffix": string (optional)
}

Key Features:
- Combines two datasets (arrays of objects) based on matching key(s)
- Supports multiple key matching using lists
- Allows different merge types (similar to pandas merge)
- Optional suffixes to distinguish columns from each input
- Takes outputs from previous workflow steps as inputs
- Requires unique operation ID

Merge Types:
- inner: Only return records with matching keys in both datasets
- left: Return all records from left dataset, with matched right dataset fields
- right: Return all records from right dataset, with matched left dataset fields
- outer: Return all records from both datasets, filling missing fields with null

Example 1: Basic Inner Merge
Left Input (Users):
[
  {"id": 1, "name": "John Doe", "department": "Sales"},
  {"id": 2, "name": "Jane Smith", "department": "Marketing"},
  {"id": 3, "name": "Bob Wilson", "department": "Engineering"}
]

Right Input (Salaries):
[
  {"id": 1, "salary": 50000},
  {"id": 2, "salary": 55000},
  {"id": 4, "salary": 60000}
]

Operation:
{
  "id": "mergeUserSalaries",
  "operation": "MERGE",
  "left_input": "getUsers",
  "right_input": "getSalaries",
  "left_on": ["id"],
  "right_on": ["id"],
  "merge_type": "inner",
  "left_suffix": "_from_users",
  "right_suffix": "_from_salaries"
}

Output:
[
  {
    "id_from_users": 1, 
    "name": "John Doe", 
    "department": "Sales",
    "id_from_salaries": 1,
    "salary": 50000
  },
  {
    "id_from_users": 2, 
    "name": "Jane Smith", 
    "department": "Marketing",
    "id_from_salaries": 2,
    "salary": 55000
  }
]

Example 2: Multiple Key Merge
Left Input:
[
  {"id": 1, "region": "North", "product": "A"},
  {"id": 2, "region": "South", "product": "B"},
  {"id": 3, "region": "East", "product": "C"}
]

Right Input:
[
  {"id": 1, "region": "North", "sales": 1000},
  {"id": 2, "region": "South", "sales": 1500},
  {"id": 4, "region": "West", "sales": 2000}
]

Operation:
{
  "id": "mergeRegionProducts",
  "operation": "MERGE",
  "left_input": "getProducts",
  "right_input": "getSales",
  "left_on": ["id", "region"],
  "right_on": ["id", "region"],
  "merge_type": "left"
}

Implementation Notes:
- 'left_on' and 'right_on' must have the same length
- Matching is performed across all specified keys