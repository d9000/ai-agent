# AI Agent — Multi-Agent Orchestration Framework

A production-ready Rails 7.1 system for LLM-powered task automation using three cooperating agents: **Planner**, **Executor**, and **Validator**.

## Stack

- **Ruby 3.3 / Rails 7.1** — application framework
- **PostgreSQL** — primary datastore (workflows, executions, memory)
- **SolidQueue** — background job processing
- **ActionCable** — real-time execution streaming
- **OpenAI GPT-4o** — planner and executor LLM (`ruby-openai`)
- **Anthropic Claude** — validator LLM (`anthropic`)
- **pgvector** — semantic long-term memory search
- **Redis** — short-term memory cache and ActionCable pub/sub

## How it works

```
POST /api/v1/workflows/:id/execute
        │
        ▼
PlannerAgent       → decomposes objective into a dependency-ordered step plan
ExecutorAgent      → runs each step, calls tools, produces structured output
ValidatorAgent     → checks correctness, schema, and safety for each step
        │
        ▼
ActionCable        → streams real-time events to the client
```

Steps retry automatically with exponential backoff. Failed steps trigger configurable fallback strategies (retry, skip, replan, escalate). The orchestrator recovers from crashes using persisted state.

## Quick start

```bash
bundle install
rails db:create db:migrate db:seed
rails credentials:edit   # add openai_api_key, anthropic_api_key
rails server
# in another terminal:
bundle exec rails solid_queue:start
```

```bash
curl -X POST http://localhost:3000/api/v1/workflows/1/execute \
  -H "Authorization: Bearer $API_AUTH_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"objective": "...", "input": {}}'
```

## Architecture

See **[ORCHESTRATION.md](ORCHESTRATION.md)** for the full specification:
agent prompts, tool interface, memory system, self-healing logic, guardrails, observability, and API contracts.

## Environment variables

| Variable | Description |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `REDIS_URL` | Redis connection string |
| `LLM_DEFAULT_MODEL` | OpenAI model (default: `gpt-4o`) |
| `LLM_VALIDATOR_MODEL` | Anthropic model for validation |
| `ALLOWED_TOOL_DOMAINS` | Comma-separated HTTP tool allowlist |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | OpenTelemetry collector endpoint |

Secrets (`openai_api_key`, `anthropic_api_key`, `api_auth_token`, `webhook_signing_secret`) are stored in Rails encrypted credentials.
