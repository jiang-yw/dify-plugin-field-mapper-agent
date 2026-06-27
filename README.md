# Field Mapper Agent

Field Mapper Agent is a Dify agent strategy plugin for structured information collection. It helps applications ask users for required fields over multiple turns, extract answers from natural language, and normalize extracted values through deterministic mappings.

This is useful when downstream workflows require exact codes or canonical values, while end users may provide aliases, abbreviations, or natural-language labels.

## Source Repository

https://github.com/jiang-yw/dify-plugin-field-mapper-agent

## Features

- Task-oriented information collection
- Multi-turn conversation state persistence
- LLM-based answer validation and extraction
- Multi-field extraction from a single user message
- Deterministic value normalization with `mapping`
- Structured option mapping with `options`, `label`, and `aliases`
- General multi-turn dialogue strategy for instruction-guided conversations

## Strategies

### TOD

The TOD strategy collects structured fields from users. Each field can define a question, whether it is required, and optional mapping rules.

Parameters:

- `model`: LLM model configuration
- `information_schema`: JSON string describing the fields to collect
- `query`: User input
- `storage_key`: Unique key for storing dialogue state

Example schema:

```json
{
  "fields": [
    {
      "name": "region_code",
      "question": "Which service region do you want to query?",
      "required": true,
      "mapping": {
        "North Region": "REG-001",
        "North Service Area": "REG-001",
        "South Region": "REG-002",
        "South Service Area": "REG-002"
      }
    },
    {
      "name": "business_type",
      "question": "Which service category do you want to query?",
      "required": true,
      "options": [
        {
          "value": "SVC-102 Standard Service Request",
          "label": "Standard Service Request",
          "aliases": [
            "standard request",
            "basic service",
            "102"
          ]
        }
      ]
    },
    {
      "name": "start_time",
      "question": "What is the start date? Use yyyyMMdd.",
      "required": true
    },
    {
      "name": "end_time",
      "question": "What is the end date? Use yyyyMMdd.",
      "required": true
    }
  ]
}
```

When a user enters `North Service Area`, the stored value for `region_code` will be `REG-001`. When a user enters `standard request`, the stored value for `business_type` will be the canonical value from the matching option.

### MTD

The MTD strategy follows a dialogue instruction and manages multi-turn branching conversations.

Parameters:

- `model`: LLM model configuration
- `instruction`: Dialogue instruction
- `query`: User input
- `storage_key`: Unique key for storing dialogue state

## Workflow Usage

A common workflow pattern is:

1. Use a question classifier to select a business branch.
2. Call this plugin's TOD strategy in each branch.
3. Pass a branch-specific `information_schema`.
4. Use the final JSON output as normalized parameters for downstream nodes or tools.

Use a stable `storage_key`, such as a conversation ID plus branch name, so each branch keeps its own collection state.

## Notes

- Mapping is applied after LLM extraction, so normalization does not rely only on model behavior.
- If no mapping matches, the original extracted value is preserved.
- For exact downstream integrations, include all known aliases in `mapping` or `options`.

## Changelog

### 0.0.6

- Renamed the plugin to Field Mapper Agent.
- Added deterministic value mapping support for collected fields.
- Updated metadata for marketplace submission.

### 0.0.5

- Added schema support for `description`, `mapping`, and `options`.
- Added post-extraction normalization.

### 0.0.4

- Updated SDK dependency range.

### 0.0.3

- Added MTD strategy support.
- Fixed data type validation issues.
- Added support for schema JSON without leading spaces.
