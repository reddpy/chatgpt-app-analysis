# Master telemetry / feature map

Two independent telemetry layers ship in the app. Merged into
`MASTER-TELEMETRY-MAP.txt` (1,867 entries = 1,466 JS + 398 Rust + headers).

| Layer | Source | Namespace | Sink |
|---|---|---|---|
| **Desktop product events** | `app.asar` JS (`logProductEvent`) | `CODEX_*` enums | Statsig / Segment-CES / Sentry / OTel |
| **Core agent events** | Rust `codex` (`analytics` + `otel` crates) | `codex.*` dotted | OTel exporter / analytics backend |

They cover different things: JS = **UI/product** interactions; Rust = **agent
runtime** behavior. Together they're a near-complete map of what the product
measures.

## Rust core event families (`codex.*`, 398)
```
exec_server 29   turn 24   mcp 24   windows_sandbox 21
sqlite 13   guardian_v2 12   apps 12   thread 11
plugins 11   memory 11   goal 11   skills 9
rollout_compression 9   guardian 8   voice 7   rollout 6
multi_agent 6   hooks 6   websocket 5   tool 5   task 5
network_proxy 5   rollout_migration 5   remote_models 4
db 4   code_mode 4   cloud_config_bundle 4   sse_event 3
```
Notable specifics:
- `codex.exec_server.*` — `process_start` (program/args/env/cwd), `http_request`
  (method/host/port/status), `sandbox_denied`, `remote.noise.*`, `rendezvous.connect`.
- `codex.guardian_v2.classification*` — `risk_level`, `authorization`, `outcome`.
- `codex.turn.token_usage`, `codex.turn.cost_microusd`, `codex.conversation.turn.count`.
- `codex.memory*`, `codex.goal*`, `codex.plugins*`, `codex.skills*`, `codex.multi_agent*`.
- `codex.rollout_compression*`, `codex.rollout_migration*`, `codex.sqlite*`.
- `codex.windows_sandbox.readiness/setup*`, `codex.network_proxy*`.
- `codex.responses_api_engine_*_ttft/_tbt.duration_ms`, `codex.api_request*`.

## JS product event families (`CODEX_*`, 1,466) — see PRODUCT-EVENT-TAXONOMY.md
```
REALTIME 162  PLUGINS 115  BROWSER 81  CHATGPT 64  AUTOMATION 58
REMOTE 52  ARTIFACT 51  CONVERSATIONAL 47  QUICK 40  AVATAR 34
DICTATION 32  APPGEN 29  MICRO 28  SHARED 27  SKETCH 26  PROFILE 24
PRIMARY 23  COMPUTER 23  ELECTRON 19  REFERRAL 18  ROSALIND 17
SITES 17  ONBOARDING 17  SAFETY 16  GOOGLE 16  REPLAY 15
GENERATED 15  THREAD 14  PROJECT 13  CHROME 13  WINDOWS 12
MINI 12  LEARNING 12  INLINE 12  FILE 12  VISUAL 10
```

## Cross-cutting view — what the two layers together reveal
- **Agent runtime health**: turn/token/cost, exec-server process+HTTP telemetry,
  guardian risk scoring, rollout/sqlite state (Rust side).
- **Product surface & engagement**: every UI interaction, feature entry point,
  onboarding funnel, error taxonomy (JS side).
- **Roadmap**: unreleased surfaces appear as enums before the UI ships
  (`ROSALIND` enrollment, `SKETCH`, `REFERRAL`, `LEARNING`,
  `CONVERSATIONAL_ONBOARDING`, `APPGEN`).
- **Privacy-relevant payloads**: the Rust layer reports **process args/env/cwd**,
  HTTP method/host/status, git repo/branch, tool/MCP call names, token usage.
  The JS layer reports feature usage + error states (not prompt text).

## Where it goes
- Rust: OTel exporter (`OTEL_TRACES_*`, `CODEX_OTEL_*`) + the analytics backend.
- JS: Statsig (`ab.chatgpt.com/v1`), Segment/CES (`/ces/v1/rgstr`,
  `/v1/log_event`), Sentry (breadcrumbs), Datadog RUM (`/telemetry/intake`).

## Files
- `MASTER-TELEMETRY-MAP.txt` — merged list
- `PRODUCT-EVENT-TAXONOMY.txt` / `PRODUCT-EVENT-TAXONOMY.md` — JS side
- `/tmp`-independent Rust list came from `codex-src` + the binary strings
