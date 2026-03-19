# Dispatching {{agent_name}}

## Quick Dispatch
Use the Agent tool with:
- **prompt:** Read `{{prompt_path}}` for your full instructions. [Task description here.]
- **subagent_type:** general-purpose
- **description:** {{agent_name}} [brief task]

## Full Dispatch Template
````
Read {{prompt_path}} for your full instructions.

Your task: [describe the specific work]

Input: [paths to relevant files or context]

Write your output to: {{output_path}}/[output-file]
````

## What to Provide
{{input_requirements}}

## Example
````
Read {{prompt_path}} for your full instructions.

Your task: {{example_task}}

Input: {{example_input}}

Write your output to: {{output_path}}/{{example_output_file}}
````
