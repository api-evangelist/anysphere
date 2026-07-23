---
name: Create and monitor a Cursor Cloud Agent
description: Launch a Cloud Agent on a repository, watch its run to completion, and collect the artifacts it produces.
api: openapi/anysphere-cloud-agents-openapi-original.yml
operations: [createAgent, getRun, streamRun, listArtifacts, downloadArtifact]
---

# Create and monitor a Cursor Cloud Agent

Use the Cursor Cloud Agents API (`https://api.cursor.com`) to run an autonomous coding
agent against a repository and retrieve its output.

## Auth
Send your Cursor Dashboard API key as HTTP Basic (key as username, empty password) or as
`Authorization: Bearer <key>`. Confirm the key with `getApiKeyInfo` (`GET /v1/me`) if unsure.

## Steps
1. **Create the agent** — `createAgent` (`POST /v1/agents`). Provide the prompt, the target
   `source` repository (see `listRepositories`), and optionally a `model` (see `listModels`).
   The response returns the durable `agent` (id `bc-*`) and the initial `run` (id `run-*`).
2. **Watch the run** — either poll `getRun` (`GET /v1/agents/{id}/runs/{runId}`) until the run
   status is terminal, or subscribe to `streamRun` (`GET /v1/agents/{id}/runs/{runId}/stream`)
   for live SSE events. Streams are resumable with `Last-Event-ID`; a `410 stream_expired`
   means the retention window elapsed — fall back to `getRun`.
3. **Collect output** — `listArtifacts` (`GET /v1/agents/{id}/artifacts`), then
   `downloadArtifact` (`GET /v1/agents/{id}/artifacts/download`) to get a presigned S3 URL.

## Rules
- Pagination is cursor-based: pass `limit` (default 20, max 100) and the `cursor` from the
  previous response; stop when `nextCursor` is absent.
- On `429`, honor `Retry-After` / `X-RateLimit-Reset`. Distinguish `rate_limit_exceeded`
  (slow down) from `usage_limit_exceeded` (plan/usage ceiling — do not just retry).
- Errors are `{ "error": { "code", "message" } }` (not problem+json). Handle `agent_busy`
  and `agent_archived` (409) before sending new work.
