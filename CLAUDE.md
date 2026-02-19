# CLAUDE.md — N8N Workflows Repository

## Repository Overview

This repository stores [n8n](https://n8n.io/) workflow automation definitions. n8n is a self-hostable, node-based workflow automation tool that connects APIs, services, and data sources without writing custom integration code.

Workflows are exported and stored as **JSON files** from an n8n instance, enabling version control, collaboration, and reuse across environments.

---

## Repository Structure

```
N8N_Workflows-/
├── CLAUDE.md                  # This file
├── workflows/                 # Main workflow JSON files (by category or feature)
│   ├── <category>/
│   │   └── <workflow-name>.json
│   └── <workflow-name>.json
├── credentials/               # Credential templates (NO real secrets — placeholders only)
│   └── <service>-template.json
├── docs/                      # Human-readable documentation per workflow
│   └── <workflow-name>.md
└── README.md                  # Project overview for humans
```

> **Note:** The exact directory layout may evolve. Keep workflows grouped logically by integration, department, or function.

---

## What is an n8n Workflow JSON?

Each `.json` file is a complete workflow export from n8n containing:

- **`nodes`** — array of node objects (each node is a step in the automation)
- **`connections`** — wiring between nodes (what feeds into what)
- **`settings`** — execution settings (timeout, error handling, timezone)
- **`staticData`** — persisted state across executions (if any)
- **`meta`** — workflow metadata (n8n version, template ID)

Example minimal structure:

```json
{
  "name": "My Workflow",
  "nodes": [...],
  "connections": {...},
  "settings": {
    "executionOrder": "v1"
  },
  "staticData": null
}
```

---

## Key Conventions

### File Naming

- Use **kebab-case** for all filenames: `slack-alert-on-error.json`
- Prefix with a category if no subdirectory is used: `crm-lead-sync.json`
- Avoid spaces and special characters in file names

### Workflow Naming (inside JSON)

- Workflow `"name"` field should be **human-readable** and descriptive
- Use title case: `"Slack Alert on Error"`
- Keep names under 60 characters

### Credentials

- **Never commit real credentials, API keys, tokens, or passwords**
- Replace sensitive values in exported JSON with placeholders like `"<YOUR_API_KEY>"` or `"REPLACE_ME"`
- Provide a `credentials/` template file explaining what each field requires
- Use n8n's built-in credential manager on the server; only template schemas live here

### Node IDs

- n8n generates UUIDs for each node (`"id"` fields inside nodes)
- Do not manually edit node IDs — they are used for internal connection references
- When merging or deduplicating, keep IDs stable

### Version Control Practices

- **One workflow per file** — never bundle multiple unrelated workflows into one JSON
- Export from n8n via `Workflow → Download` or the n8n CLI/API
- Import to n8n via `Workflow → Import from file` or the n8n CLI/API
- Always test a workflow in a **staging n8n instance** before adding to this repo

---

## Development Workflow

### Adding a New Workflow

1. Build and test the workflow in your n8n instance
2. Export it: `Workflow menu → Download`
3. Strip or replace any credentials/secrets in the JSON
4. Save to `workflows/<category>/<workflow-name>.json`
5. Add a brief doc in `docs/<workflow-name>.md` describing:
   - Purpose
   - Trigger type (webhook, schedule, manual)
   - Required credentials/environment variables
   - Expected inputs and outputs
6. Open a PR for review

### Updating an Existing Workflow

1. Import the existing JSON into your n8n instance
2. Make changes and test
3. Re-export and replace the file in the repo
4. Update the corresponding `docs/` file if behavior changed
5. Commit with a descriptive message (see below)

### Deleting a Workflow

- Remove the `.json` file and its `docs/` counterpart
- Note in the commit message why it was removed (deprecated, replaced by another, etc.)

---

## Git Commit Conventions

Use clear, imperative commit messages:

```
add: slack-alert-on-error workflow
update: crm-lead-sync - add retry logic on HTTP 429
fix: broken connection in data-enrichment workflow
remove: legacy-hubspot-sync (replaced by hubspot-v2-sync)
docs: add documentation for stripe-payment-capture
```

Prefix options: `add`, `update`, `fix`, `remove`, `docs`, `refactor`, `chore`

---

## Importing & Exporting Workflows (n8n CLI)

If the n8n CLI is available:

```bash
# Export a workflow by ID
n8n export:workflow --id=<workflow-id> --output=workflows/<name>.json

# Import a workflow
n8n import:workflow --input=workflows/<name>.json

# Export all workflows
n8n export:workflow --all --output=workflows/
```

If using the n8n REST API:

```bash
# Export via API
curl -H "X-N8N-API-KEY: $N8N_API_KEY" \
  "http://localhost:5678/api/v1/workflows/<id>" \
  | jq . > workflows/<name>.json

# Import via API
curl -X POST \
  -H "X-N8N-API-KEY: $N8N_API_KEY" \
  -H "Content-Type: application/json" \
  -d @workflows/<name>.json \
  "http://localhost:5678/api/v1/workflows"
```

---

## AI Assistant Guidelines

When analyzing or modifying workflows in this repository:

### Reading Workflows

- Parse the `nodes` array to understand the automation steps
- Follow `connections` to trace the data flow from trigger to final action
- Check `settings.executionOrder` — `"v1"` is the modern execution model
- Identify the trigger node (type usually contains `Trigger`, e.g., `n8n-nodes-base.webhookTrigger`, `n8n-nodes-base.scheduleTrigger`)

### Modifying Workflows

- Preserve all existing `"id"` fields on nodes and the workflow itself
- Do not reorder the `nodes` array — positions are cosmetic but ordering can matter
- Keep `connections` consistent with any node additions/removals
- Validate JSON is well-formed before committing (`jq . <file.json>`)

### Creating New Workflows

- Start from an exported n8n workflow template, not from scratch
- Follow file naming and workflow naming conventions above
- Never invent credential values — use placeholder strings

### Security

- Scan for accidental secrets before every commit:
  ```bash
  grep -rE "(api_key|apikey|password|secret|token|Bearer)" workflows/ --include="*.json"
  ```
- If secrets are found, replace with `"REPLACE_ME"` and note in docs what value is needed
- Do not add `.env` files or credential exports to this repo

### JSON Validation

Before committing any workflow JSON:

```bash
# Validate all workflow JSON files
for f in workflows/**/*.json; do
  jq empty "$f" && echo "OK: $f" || echo "INVALID: $f"
done
```

---

## Environment Variables / Configuration

If this repository is used with CI/CD or scripting, the following environment variables may be referenced:

| Variable | Description |
|---|---|
| `N8N_API_KEY` | API key for authenticating with the n8n instance |
| `N8N_BASE_URL` | Base URL of the n8n instance (e.g., `https://n8n.example.com`) |
| `N8N_ENCRYPTION_KEY` | Encryption key for n8n credentials (server-side only) |

These should be set in your environment or CI secrets — **never committed to this repository**.

---

## Common Node Types Reference

| Node Type | Purpose |
|---|---|
| `n8n-nodes-base.webhook` | HTTP webhook trigger |
| `n8n-nodes-base.scheduleTrigger` | Cron/interval-based trigger |
| `n8n-nodes-base.httpRequest` | Make HTTP requests to any API |
| `n8n-nodes-base.set` | Transform/set field values |
| `n8n-nodes-base.if` | Conditional branching |
| `n8n-nodes-base.switch` | Multi-branch routing |
| `n8n-nodes-base.merge` | Combine data from multiple branches |
| `n8n-nodes-base.code` | Run custom JavaScript/Python |
| `n8n-nodes-base.noOp` | Passthrough / placeholder node |
| `n8n-nodes-base.errorTrigger` | Catch workflow execution errors |

---

## Resources

- [n8n Documentation](https://docs.n8n.io/)
- [n8n Node Library](https://n8n.io/integrations/)
- [n8n Community Forum](https://community.n8n.io/)
- [n8n Workflow Templates](https://n8n.io/workflows/)
- [n8n REST API Reference](https://docs.n8n.io/api/)
- [n8n CLI Reference](https://docs.n8n.io/hosting/cli-commands/)
