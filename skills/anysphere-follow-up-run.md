---
name: Send a follow-up prompt and reconcile agent usage
description: Continue an existing Cloud Agent with a new run, track its token usage, and archive it when done.
api: openapi/anysphere-cloud-agents-openapi-original.yml
operations: [listAgents, createRun, listRuns, getRun, getAgentUsage, cancelRun, archiveAgent]
---

# Send a follow-up prompt and reconcile agent usage

Continue work on an existing Cursor Cloud Agent and account for what it consumed.

## Auth
Cursor Dashboard API key via HTTP Basic or `Authorization: Bearer <key>`.

## Steps
1. **Find the agent** — `listAgents` (`GET /v1/agents`, newest first) or use a known `bc-*` id.
2. **Send a follow-up** — `createRun` (`POST /v1/agents/{id}/runs`) with the next prompt. This
   creates a new `run-*` on the same durable agent.
3. **Track runs** — `listRuns` (`GET /v1/agents/{id}/runs`) and `getRun` for status/results.
4. **Cancel if needed** — `cancelRun` (`POST /v1/agents/{id}/runs/{runId}/cancel`); a
   `409 run_not_cancellable` means the run already finished.
5. **Reconcile usage** — `getAgentUsage` (`GET /v1/agents/{id}/usage`) for token usage.
6. **Wind down** — `archiveAgent` (`POST /v1/agents/{id}/archive`) when the agent is idle;
   `unarchiveAgent` to bring it back.

## Rules
- One prompt = one run; the agent is durable across runs. Do not create a new agent to
  continue a conversation.
- Respect rate-limit headers on `429`; back off on `agent_busy` (409) rather than hammering.
- Cursor pagination (`limit`/`cursor` → `nextCursor`) applies to `listAgents` and `listRuns`.
