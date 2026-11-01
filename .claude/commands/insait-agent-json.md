---
description: Build or validate a Conversation Flow Agent JSON for the Insait platform. Pass a file path to validate/fix, or describe the agent to build from scratch. Covers full schema, node types, exits, variables, and all import failure modes.
argument-hint: [/path/to/agent.json | "describe the agent you want to build"]
---

You are an expert in the Insait AI Agent Platform's Conversation Flow Agent JSON format. Your job is to build a valid agent JSON or validate and fix an existing one so it imports without errors.

Parse $ARGUMENTS:
- File path (starts with `/` or `./` or ends with `.json`): read the file, validate it against ALL rules below, output a corrected file + list of every fix made.
- Description of an agent: build a complete, valid, importable agent JSON from scratch.
- Empty: explain the format and ask what the user needs.

---

## What You Are Building

A Conversation Flow Agent JSON exported from (or importable into) the Insait platform. This file contains the agent configuration, the entire flow definition (nodes + edges), custom tools, and optionally testing data.

The platform imports this via `POST /api/v1/agents/import`. Validation happens server-side and will reject the file with a 400 error if any rule below is violated.

---

## Top-Level Structure

```json
{
  "metadata": {
    "export_version": "2.0",
    "exported_at": "2025-01-15T10:30:00Z",
    "exported_by": "user@org.com",
    "validation_status": "success",
    "validation_errors": [],
    "validation_warnings": []
  },
  "agent": { /* agent metadata — name, channel, LLM settings, etc. */ },
  "flow_definition": { /* the entire flow: global settings, nodes, edges, variables */ },
  "tools": [ /* custom API tool definitions */ ],
  "filler_sentences": [],
  "nikud_replacements": [],
  "kb_references": [ { "kb_id": "uuid", "name": "string" } ]
}
```

---

## Agent Object (required fields)

```json
"agent": {
  "name": "Agent Name (required)",
  "channel": "chat | voice (required)",
  "agent_mode": "builder (required for flow agents)",
  "agent_language": "en-US",
  "is_active": true,
  "is_live": false,
  "llm_provider": "openai | claude | gemini",
  "llm_model": "gpt-4o | claude-opus-4-5 | gemini-2.0-flash",
  "llm_temperature": 0.0,
  "system_prompt": "You are a helpful assistant..."
}
```

---

## Flow Definition Object

```json
"flow_definition": {
  "id": "uuid",
  "name": "Flow Name",
  "version": 1,
  "channel": "chat | voice",

  "global_settings": {
    "system_prompt": "You are... (required)",
    "llm_provider": "openai",
    "llm_model": "gpt-4o",
    "temperature": 0.0,
    "greeting_message": "Hi! How can I help?",
    "first_speaker": "agent | user"
  },

  "variables": [ /* VariableDefinition array — see Variables section */ ],

  "tools": {
    "global_tools": ["uuid-string-1", "uuid-string-2"],
    "built_in_tools": {
      "transfer_to_human": false,
      "end_call": false
    }
  },

  "voice_settings": { /* REQUIRED when channel = "voice" — causes 400 if missing */ },

  "flow": {
    "start_node_id": "node-start (MUST match a key in nodes)",
    "nodes": {
      "node-start": { /* start node */ },
      "node-collect-contact": { /* collect node */ },
      "node-conversation": { /* conversation node */ },
      "node-end": { /* end node */ }
    },
    "exits": [ /* all edges between nodes */ ]
  }
}
```

**⚠️ CRITICAL: `voice_settings` is REQUIRED for voice channel agents — causes HTTP 400 if missing**

When `channel = "voice"`, `flow_definition` must contain a `voice_settings` object. Use this block for Hebrew (he-IL) agents — these are the exact values from production:

```json
"voice_settings": {
  "tts_provider": "deepdub",
  "tts_voice_id": "11111111-1111-1111-1111-111111111111",
  "tts_voice_name": "Heather Long",
  "tts_model": "insait3",
  "tts_locale": "he-IL",
  "tts_speaking_rate": 1.2,
  "tts_tempo": null,
  "tts_variance": null,
  "tts_temperature": null,
  "tts_seed": null,
  "voice_instructions": null,
  "stt_provider": "soniox",
  "stt_model": null,
  "stt_language": "he-IL",
  "vad_stop_secs": 0.2,
  "vad_confidence": 0.7,
  "filler_sentences": [],
  "nikud_replacements": []
}
```

For English agents use `tts_provider: "elevenlabs"`, `tts_voice_id: "21m00Tcm4TlvDq8ikWAM"`, `stt_provider: "deepgram"`, `stt_language: "en-US"`.

---

**⚠️ CRITICAL: `flow_definition.tools.global_tools` must be a plain list of UUID strings** — the IDs of tools defined in the top-level `tools` array. When exporting v2 agents, the exporter may embed full tool objects here instead. Always replace any objects with their `original_id` string value:

```json
// ✅ CORRECT
"global_tools": ["22222222-2222-2222-2222-222222222222", "33333333-3333-3333-3333-333333333333"]

// ❌ WRONG — causes 400 on import
"global_tools": [{ "original_id": "22222222-...", "name": "Fetch User Data", ... }]
```

---

## Node Types

### START NODE
```json
{
  "id": "node-start",
  "type": "start",
  "name": "Greeting",
  "data": {
    "greeting_message": "Hi! I'm Jess. I can help you...",
    "skip_if_user_starts": true,
    "router_mode": false
  },
  "exits": [],
  "position": { "x": 0, "y": 0 }
}
```

### CONVERSATION NODE
```json
{
  "id": "node-conv-main",
  "type": "conversation",
  "name": "Main Conversation",
  "data": {
    "prompt": "Help the user with their request.",
    "use_agent_prompt": true,
    "tools": null,
    "kb_mode": null
  },
  "exits": [],
  "position": { "x": 300, "y": 0 }
}
```

### COLLECT NODE
```json
{
  "id": "node-collect-contact",
  "type": "collect",
  "name": "Collect Contact Info",
  "data": {
    "field_names": ["first_name", "last_name", "email", "phone"],
    "prompt": "I'll need a few details from you.",
    "validate_fields": false,
    "field_validations": {
      "email": [{ "type": "builtin", "validation_type": "email" }],
      "phone": [{ "type": "builtin", "validation_type": "phone", "params": { "country": "US" } }]
    }
  },
  "exits": [],
  "position": { "x": 600, "y": 0 }
}
```
**Rule:** `field_names` must reference variable names defined in the `variables` array. Do NOT use the legacy `fields` array in new agents.

### END NODE
```json
{
  "id": "node-end",
  "type": "end",
  "name": "Goodbye",
  "data": {
    "end_message": "Thank you! Have a great day.",
    "save_transcript": true
  },
  "exits": [],
  "position": { "x": 900, "y": 0 }
}
```

### API NODE (call an external tool automatically)
```json
{
  "id": "node-api-call",
  "type": "api",
  "name": "Call External API",
  "data": {
    "tool_id": "tool-uuid (must exist in top-level tools array)",
    "parameter_mapping": {
      "customer_id": "{{customer_id}}",
      "amount": "{{loan_amount}}"
    },
    "result_variable": "api_result",
    "timeout_seconds": 30
  },
  "exits": [],
  "position": { "x": 900, "y": 0 }
}
```

### CODE NODE (run custom Python logic)
```json
{
  "id": "node-suburb-lookup",
  "type": "code",
  "name": "Suburb Lookup",
  "data": {
    "code": "# Python code here\nresult = input.get('suburb_name', '')\nreturn {'suburb_id': '123', 'status': 'resolved'}",
    "language": "python",
    "output_mode": "single",
    "output_variable": "suburb_result",
    "timeout_seconds": 30
  },
  "position": { "x": 900, "y": 0 }
}
```
- `input` is available as a dict containing all current conversation variables
- Return a dict — it is stored in `output_variable`
- `output_mode`: `"single"` (return one object) or `"stream"` (return multiple)

**⚠️ CRITICAL: Code node `code` field is limited to 100,000 characters.**
Embedding large inline datasets (e.g. a full national suburb/postcode lookup table as a zlib+base64 blob) will exceed this limit and cause a 400 on import. Options when you need large lookup data:
1. **Split across two code nodes** — partition the dataset (e.g. by state/region), have node 1 return `{"status": "try_next"}` on no-match, and route to node 2 via an expression exit
2. **Use an external API tool** instead of embedding the data inline

---

### TRANSFER TO HUMAN NODE
```json
{
  "id": "node-transfer",
  "type": "transfer_to_human",
  "name": "Transfer to Specialist",
  "data": {
    "transfer_message": "Let me connect you with a specialist.",
    "targets": [
      { "id": "uuid", "name": "Sales", "phone_number": "+15551234567", "priority": 1 }
    ]
  },
  "exits": [],
  "position": { "x": 900, "y": 300 }
}
```

---

## Exits (Edges Between Nodes)

Exits live in the top-level `flow.exits` array (NOT inside individual nodes).

```json
{
  "id": "exit-uuid",
  "name": "User chose savings",
  "source_node_id": "node-start",
  "target_node_id": "node-collect-contact",
  "priority": 0,
  "condition": {
    "type": "always | expression | llm | result"
  }
}
```

### ALWAYS (default/fallback — fires unconditionally)
```json
{ "type": "always" }
```

### EXPRESSION (evaluate a variable value)
```json
{
  "type": "expression",
  "expression": "{{intent}} == 'savings'"
}
```
Supported operators: `==`, `!=`, `>`, `<`, `>=`, `<=`, `in`
Logical: `AND`, `OR` (uppercase)
Truthy check: `{{variable_name}}` or `!{{variable_name}}`
**Rule:** Expression MUST have an operator. `{{var}}` alone is falsy/truthy check only.

### LLM (ask the LLM to decide)
```json
{
  "type": "llm",
  "prompt": "Did the user agree to the terms?",
  "context_window": 3
}
```
**Rule:** LLM condition `prompt` must be ≤ 1000 characters. If you need to reference a long list (e.g. distributor names), keep the list in the system prompt and reference it from the exit condition prompt: "…or mentions any name from the distributor list in the system prompt."

### RESULT (after API node — success or error path)
```json
{ "type": "result", "result": "success | error" }
```

**Priority:** Lower number = evaluated first. Use priority to control which exit fires when multiple conditions could match. Always put the `always` (fallback) exit last (highest priority number).

---

## Variables

Variables are defined once at the top level of `flow_definition.variables`. Collect nodes reference them by name in `field_names`.

```json
{
  "name": "first_name",
  "type": "string | number | boolean | date | enum | list | object | document",
  "description": "The user's first name",
  "required": true,
  "persist": true,
  "source": "collect",
  "source_node_id": "node-collect-contact"
}
```

### ENUM — REQUIRES options array
```json
{
  "name": "credit_score_range",
  "type": "enum",
  "options": ["500-549", "550-599", "600-649", "650-699", "700+"],
  "description": "Bucketed credit score range"
}
```

### ⚠️ CRITICAL: `options` on string variables triggers interactive button UI

When a variable has `options` set (even on a `"type": "string"` field), the platform renders those options as interactive buttons in the collect node UI. **This breaks silently at runtime — no import error, no warning — but the LLM call hangs and the conversation gets stuck.**

| Scenario | What to do |
|----------|-----------|
| 2–5 short choices the user must pick from | Use `"type": "enum"` with `options` — buttons are intentional |
| Free-text field where LLM normalises input (e.g. state names, countries) | Set `"options": null` — let the LLM handle normalisation via the prompt |
| Long lists (10+ items, e.g. all 50 US states) | **Never use `options`** — always `null`. Large button lists cause the renderer to hang. |

**Real incident:** A `string` variable for US state registration had all 50 states in `options`. Every conversation hung silently on the state question. Fix: set `"options": null` and rely on the node prompt to normalise `"CA" → "California"`.

### VALID SOURCE VALUES
`source` must be one of: `"user"`, `"collect"`, `"tool"`, `"system"`, `"session"`
- `"collect"` — value comes from a Collect node field
- `"tool"` — value extracted from an API/tool response
- `"user"` — value injected from session data (iframe/API)
- `"system"` — value set by the agent's internal logic (LLM-inferred flags, computed values)
- `"session"` — value passed in via session context

**Never use `"infer"` — it is not a valid source value and will cause a 400 on import.**

### RESERVED NAMES — NEVER USE THESE
`user`, `system`, `assistant`, `tool`, `node`, `flow`, `turn`, `message`, `messages`, `conversation_id`, `external_id`, `stream`, `session_data`

---

## Validation Rules (for collect node fields)

```json
"field_validations": {
  "email": [{ "type": "builtin", "validation_type": "email" }],
  "phone": [{ "type": "builtin", "validation_type": "phone", "params": { "country": "US" } }],
  "zip_code": [{ "type": "builtin", "validation_type": "postal_code", "params": { "country": "US" } }],
  "dob": [{ "type": "builtin", "validation_type": "date_not_future" }],
  "loan_amount": [{ "type": "builtin", "validation_type": "number_positive" }]
}
```

All built-in types: `phone`, `national_id`, `postal_code`, `iban`, `vat_number`, `email`, `url`, `date_format`, `time_format`, `credit_card`, `date_not_future`, `date_not_past`, `date_range`, `number_range`, `number_positive`, `number_integer`, `not_empty`, `min_length`, `max_length`, `regex`, `one_of`

---

## Tools (Custom API Integrations)

The top-level `tools` array holds full tool definitions. Each tool has an `execution_config` with a `request_chain` array containing the actual HTTP call(s).

```json
"tools": [
  {
    "original_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "name": "eCalc Get Offers",
    "description": "Submit loan data to eCalc API and return refinancing offers.",
    "category": "api",
    "type": "http_api",
    "function_definition": {
      "name": "ecalc_get_offers",
      "description": "Submit full loan application data and return offers.",
      "parameters": {
        "type": "object",
        "required": ["query"],
        "properties": {
          "query": { "type": "string", "description": "Trigger parameter" }
        }
      }
    },
    "execution_config": {
      "method": "POST",
      "endpoint": "/path/to/api",
      "headers": { "Content-Type": "application/json" },
      "body": "{\"field\": \"{{variable}}\"}",
      "body_format": "json",
      "query_params": {},
      "request_chain": [
        {
          "url": "https://api.example.com/path/to/api",
          "name": "API Call Name",
          "method": "POST",
          "headers": { "Content-Type": "application/json" },
          "body": "{\"field\": \"{{variable}}\"}",
          "body_format": "json",
          "auth": { "type": "none" },
          "query_params": {},
          "mockConfig": { "enabled": false, "delay_ms": 500, "response": "{}" },
          "errorConfig": {
            "on_error": "fail",
            "on_timeout": "fail",
            "retry_count": 0,
            "retry_delay_ms": 1000,
            "status_handlers": [],
            "timeout_seconds": 15,
            "fallback_response": ""
          },
          "extractions": [
            {
              "variable_name": "api_result_field",
              "extraction_type": "json_path",
              "response_path": "$.fieldName"
            }
          ]
        }
      ]
    },
    "instance_config": {},
    "enabled": true,
    "order_index": 0,
    "custom_instructions": null,
    "trigger_messages": null,
    "disable_interruptions": false,
    "expects_response": true,
    "execution_mode": "sync",
    "response_timeout_secs": 15,
    "mock_config": null,
    "assignments": null,
    "is_system_tool": false,
    "system_tool_type": null,
    "llm_direct_params": false
  }
]
```

### ⚠️ CRITICAL: `body` Must Always Be a String, Never an Object

Inside every `request_chain[]` step, the `body` field **must be a JSON string**, not a parsed object or array. The tool configuration editor (`JsonEditor`) only knows how to display strings — if `body` is a dict or list, the UI crashes with a white screen the moment you try to open the tool.

```json
// ✅ CORRECT — body is a JSON string
"body": "{\"type\": \"enroll\", \"data\": {\"firstName\": \"{{first_name}}\"}}"

// ✅ CORRECT — multi-line string (produced by the UI's text editor)
"body": "{\n  \"type\": \"enroll\",\n  \"data\": {\n    \"firstName\": \"{{first_name}}\"\n  }\n}"

// ✅ CORRECT — GET request with no body
"body": "{}"

// ✅ CORRECT — form-encoded body is a plain string
"body": "grant_type=client_credentials&client_id=myapp&client_secret=abc123"

// ❌ WRONG — body is an object (causes white-screen crash in the tool editor)
"body": { "type": "enroll", "data": { "firstName": "{{first_name}}" } }

// ❌ WRONG — body is an array (same crash)
"body": [{ "id": "{{id}}" }]
```

**When building JSON programmatically**, always call `JSON.stringify(bodyObject, null, 2)` (JS) or `json.dumps(body_dict, indent=2)` (Python) before assigning to `body`. The platform's backend execution normalises dict bodies at runtime so the tool still *executes* correctly, but the UI crash is frontend-only and the fix must be in the JSON itself.

---

### ⚠️ CRITICAL: Extraction Field Names in `request_chain`

Inside `request_chain[].extractions`, the field names are **different** from what you might expect from the backend runtime schema. Use ONLY these names:

| ✅ Correct (import schema) | ❌ Wrong (runtime schema — causes white screen crash) |
|---|---|
| `"variable_name": "my_var"` | `"variable": "my_var"` |
| `"response_path": "$.field"` | `"extraction_path": "$.field"` |

Using `variable` or `extraction_path` will import successfully (no 400 error) but will crash the tool editor with a white screen when you try to open the tool for editing.

**Rule:** Any `tool_id` referenced in a node's `data.tools` or an API node's `data.tool_id` MUST have a matching entry in the top-level `tools` array.

---

## Import Failure Modes (What Causes 400 Errors)

These will cause the import to fail with an error. Check every one before uploading:

| # | Rule | What to check |
|---|------|---------------|
| 1 | `start_node_id` must exist | The value of `flow.start_node_id` must match a key in `flow.nodes` |
| 2 | All exit targets must exist | Every `exit.target_node_id` must match a key in `flow.nodes` |
| 3 | All exit sources must exist | Every `exit.source_node_id` must match a key in `flow.nodes` |
| 4 | Enum variables need options | Every variable with `type: "enum"` must have a non-empty `options` array |
| 5 | Object variables need properties | Every variable with `type: "object"` must have `object_properties` array |
| 6 | List variables need item type | Every variable with `is_list: true` must have `list_item_type` |
| 7 | Reserved variable names | None of the reserved names listed above |
| 8 | Tool references must resolve | Every tool_id used in nodes must exist in top-level `tools` array |
| 9 | Expression syntax | Expressions must have an operator and balanced `{{}}` braces |
| 10 | field_names references | All names in `data.field_names` must exist in `flow_definition.variables` |
| 11 | JSON validity | The file must be valid JSON (no trailing commas, no comments) |
| 12 | agent_mode | Must be `"builder"` for flow agents (not `"simple"`) |
| 13 | Variable source values | `source` must be one of: `"user"`, `"collect"`, `"tool"`, `"system"`, `"session"`. `"infer"` is NOT valid. |
| 14 | LLM exit prompt length | `condition.prompt` on LLM exits must be ≤ 1000 characters. Move long lists to the system prompt and reference them. |
| 15 | Tool extraction field names | Inside `request_chain[].extractions`, use `"variable_name"` and `"response_path"`. Using `"variable"` or `"extraction_path"` passes import but crashes the tool editor with a white screen. |
| 16 | Variable `options` list size | Any variable with `options` set causes the collect node to render interactive buttons. Lists of 10+ items (e.g. all 50 US states) cause silent LLM hangs at runtime. Set `"options": null` for free-text fields the LLM normalises via prompt. |
| 17 | `global_tools` must be string IDs | `flow_definition.tools.global_tools` must be a `List[str]` of UUID strings. v2 exports may embed full tool objects here — replace each object with its `original_id` string. |
| 18 | Code node size limit | A code node's `data.code` field must be ≤ 100,000 characters. Inline datasets (zlib+base64 blobs, lookup tables) commonly exceed this. Split across two nodes or use an external API tool. |
| 19 | `request_chain[].body` must be a string | Every `body` field inside `request_chain` steps must be a **JSON string**, not a parsed object or array. An object body passes import silently but crashes the tool editor with a white screen. Serialize with `json.dumps(body, indent=2)` before writing the JSON. |

---

## Build Checklist (Run Before Every Export)

- [ ] `flow.start_node_id` matches a real node key
- [ ] Every node that should connect to another has a corresponding exit in `flow.exits`
- [ ] Every exit's `source_node_id` and `target_node_id` exist in `flow.nodes`
- [ ] All `field_names` in collect nodes reference variables defined in `flow_definition.variables`
- [ ] Every `enum` variable has a non-empty `options` array
- [ ] No reserved variable names used
- [ ] Every `tool_id` in nodes exists in top-level `tools` array
- [ ] Expression conditions use `==`, `!=`, `>`, `<`, `>=`, `<=`, or `in` with proper `{{var}}`
- [ ] The file is valid JSON (no trailing commas, no `// comments`)
- [ ] `agent.agent_mode` is `"builder"`
- [ ] `agent.channel` matches `flow_definition.channel`
- [ ] If `channel = "voice"`: `flow_definition.voice_settings` is present and has at minimum `tts_provider` and `tts_voice_id` — **missing this causes HTTP 400**
- [ ] At least one END node exists (flow must be able to terminate)
- [ ] Every non-end node has at least one exit
- [ ] All variable `source` values are one of: `"user"`, `"collect"`, `"tool"`, `"system"`, `"session"` (never `"infer"`)
- [ ] All LLM exit condition prompts are ≤ 1000 characters
- [ ] All `request_chain[].extractions` use `"variable_name"` (not `"variable"`) and `"response_path"` (not `"extraction_path"`)
- [ ] Every `request_chain[].body` is a **string** (not a dict or array) — serialize with `json.dumps(body, indent=2)` if building programmatically
- [ ] No `string` variable has a large `options` list (10+ items) — set `"options": null` and rely on the LLM prompt for normalisation instead
- [ ] `flow_definition.tools.global_tools` is a list of UUID strings, not a list of objects
- [ ] No code node's `data.code` exceeds 100,000 characters — split large inline datasets across two nodes or replace with an API tool

---

## Minimal Working Example (Chat Agent, 2 nodes)

```json
{
  "metadata": {
    "export_version": "2.0",
    "exported_at": "2025-01-15T10:00:00Z",
    "exported_by": "builder@org.com",
    "validation_status": "success",
    "validation_errors": [],
    "validation_warnings": []
  },
  "agent": {
    "name": "My Flow Agent",
    "channel": "chat",
    "agent_mode": "builder",
    "agent_language": "en-US",
    "is_active": true,
    "is_live": false,
    "llm_provider": "openai",
    "llm_model": "gpt-4o",
    "llm_temperature": 0.0,
    "system_prompt": "You are a helpful assistant."
  },
  "flow_definition": {
    "id": "flow-001",
    "name": "My Flow",
    "version": 1,
    "channel": "chat",
    "global_settings": {
      "system_prompt": "You are a helpful assistant.",
      "llm_provider": "openai",
      "llm_model": "gpt-4o",
      "temperature": 0.0,
      "greeting_message": "Hi! How can I help you today?",
      "first_speaker": "agent"
    },
    "variables": [],
    "flow": {
      "start_node_id": "node-start",
      "nodes": {
        "node-start": {
          "id": "node-start",
          "type": "start",
          "name": "Greeting",
          "data": { "greeting_message": "Hi! How can I help you today?", "skip_if_user_starts": true },
          "exits": [],
          "position": { "x": 0, "y": 0 }
        },
        "node-end": {
          "id": "node-end",
          "type": "end",
          "name": "Goodbye",
          "data": { "end_message": "Thank you! Have a great day.", "save_transcript": true },
          "exits": [],
          "position": { "x": 400, "y": 0 }
        }
      },
      "exits": [
        {
          "id": "exit-start-to-end",
          "name": "Continue",
          "source_node_id": "node-start",
          "target_node_id": "node-end",
          "priority": 0,
          "condition": { "type": "always" }
        }
      ]
    }
  },
  "tools": [],
  "filler_sentences": [],
  "nikud_replacements": [],
  "kb_references": []
}
```

---

## If Validating an Existing File

1. Read the file
2. Check every rule in the Import Failure Modes table
3. Check every item in the Build Checklist
4. Fix all issues found
5. Output: the corrected JSON + a summary table of every change made (field path → what was wrong → what was fixed)
