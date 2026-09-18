# Backend API map & new codenames

Extracted **488 backend endpoints** (path + HTTP method) from the renderer —
the complete server contract the app uses. Full list: `BACKEND-API-MAP.txt`.
This pass surfaced several **new internal codenames**.

## New backend namespaces (codenames)

| Namespace | # | What it appears to be |
|---|---|---|
| **aip** | 44 | "AI Platform": **connectors** (email send/unsend, GitHub, Google contacts, OAuth links) + **ledger** (financial memories, links, accounts, credit, widgets) + **olympic** (oauth) |
| **hermes** | 25 | **Agents with triggers**: `/hermes/agent/{id}`, `/hermes/agents/pinned`, `/hermes/agent/{id}/triggers/{id}`, `/hermes/agent/{id}/system-hint` — a scheduled/agent platform |
| **amphora** | 16 | **Workspaces**: members, invites, permissions, notifications, `clear_settings_cache`, `u18_graduation_unlink_setting_notices` |
| **cme** | 6 | Theme/account artwork: `/cme/account`, `/cme/claims`, `/cme/conversations/{id}/remove` |
| **hazelnuts** | 7 | Generic objects: `/hazelnuts/{hazelnut_id}` (PATCH) |
| **snorlax** | 3 | `/gizmos/snorlax/sidebar` |
| **celsius** | 1 | `/celsius/ws/user` (websocket user) |
| **pets** | 4 | `/pets/create` |
| **quicksilver / avas** | — | realtime voice: `/wham/realtime/calls?intent=quicksilver&architecture=avas` |

## Notable feature surfaces in the API
- **Ledger / finance** (`/aip/ledger/*`): financial memories, links to accounts,
  credit connection, widgets, OAuth — a personal-finance/assistant ledger.
- **Connectors** (`/aip/connectors/*`): email (`send_email`, `unsend_email`),
  GitHub installations, Google contacts search, OAuth link flows.
- **Automations** (`/automations/*`): `generate_creation_prompt`, `save`,
  `set_status`, `run`, `webhook_batch/run_now`, share/delete.
- **Memories** (`/memories/*`): CRUD + history + status.
- **File library** (`/files/library/*`): directories, files, trash, versions,
  content/download URLs, mounted search.
- **Agents/triggers** (`/hermes/*`): agent CRUD + triggers + pinned list.
- **Workspaces** (`/amphora/*`): membership + permissions + invites.
- **Enterprise/account**: `/accounts/{id}/workspace_policy`,
  `spend-controls/current-user/monthly-usage`, `remaining_balance`, seats,
  MFA (`/accounts/mfa/*`), `logout_all`, `logout-impact`.
- **Ads** (`/bazaar/*`) — see `BAZAAR-ADS.md`.
- **Beacons** (`/beacons/event`) — in-app surveys.
- **Realtime voice** (`/wham/realtime/*`, `/transcribe`, `/pronunciation/synthesize`).
- **Pets** (`/pets/create`).
- **Gizmos** (custom GPTs) incl. `/gizmos/snorlax/sidebar`,
  `/gizmos/firstparty/sidebar`, `/gizmos/bootstrap`.
- **AIP olympic debug**: `/aip/olympic/debug/test-account-connection`.

## Auth / identity surfaces
`/accounts/check/{version}`, `/accounts/optimized/check`, `/accounts/mfa_info`,
`/accounts/verified_access`, `/ios/attestation_challenge`,
`/legalapi/enabled`, `/api/auth/csrf`, `/api/auth/_log`.

## Why it's juicy
This is the **server-side product map**: unreleased/platform features
(aip/ledger finance, hermes agents+triggers, amphora workspaces, automations,
memories, file library) are all visible by endpoint, including debug endpoints
(`/aip/olympic/debug/*`) and the realtime-voice architecture codenames
(`quicksilver`, `avas`).

## Use in dev mode
- Query Devtools → inspect any of these endpoints' cached responses.
- Debug panel → `chatgpt-api` section lists live requests (target = these paths).
- Statsig map (`STATSIG-ID-MAP.md`) → the gates behind them.
