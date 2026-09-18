# End-to-end trace: "Hermes" / Workspace Agents

`hermes` is the backend namespace for **Workspace Agents** — a full **agent
builder + scheduler** inside the app (build an agent, connect tools, schedule
triggers, publish versions, chat with it).

## 1. Entry point / gating
- Sidebar nav: `sidebarElectron.workspaceAgentsRouteNavLink` → **"Agents"**
  (`/agents` route), shown when `isCapable && N === true`.
- Feature enable rule (from the store):
  ```js
  enabled: accountId != null && get(<flag Fx>) === true
  ```
  (an account flag / Statsig gate; the exact ID wasn't resolvable from the
  minified cross-module refs).
- Pins persisted per account: key `workspace-agent-pins-by-account-v1`.

## 2. UI flow
```
sidebar "Agents"
  → workspace-agent-landing            (agent grid / browse)
  → workspace-agent-detail-page        (agent detail)
       ├─ builder-view                 (edit)
       ├─ conversations                (chat with the agent)
       ├─ activity                     (runs)
       └─ published-versions
  → composer placeholder "Ask {agentName}"
  → agent-menu / duplicate-agent
```
In-conversation widgets: `widgets.hermes.*` (elicitation/tool-approval,
wait-state messages, generic response, fullscreen view).

## 3. API surface (20 endpoints, `/hermes/*`)
```
GET    /hermes/agents                         list
GET    /hermes/agents/pinned                  pinned
POST   /hermes/agent                          create
GET    /hermes/agent/{id}                     read
POST   /hermes/agent/{id}/save                save
POST   /hermes/agent/{id}/save_publish        save + publish
POST   /hermes/agent/{id}/clone               clone
POST   /hermes/agent/{id}/pin                 pin/unpin
GET    /hermes/agent/{id}/builder-view        editor payload
GET    /hermes/agent/{id}/detail-view         detail payload
GET    /hermes/agent/{id}/system-hint         system hint(s)
GET    /hermes/agent/{id}/list-connector-readiness   connector status
GET    /hermes/agent/{id}/published_versions  version history
GET    /hermes/agent/{id}/conversations       threads for this agent
GET    /hermes/agent/{id}/persistent-folder/tree            file tree
GET    /hermes/agent/{id}/persistent-folder/files/{id}/download_url
GET    /hermes/agent/{id}/triggers            list triggers
POST   /hermes/agent/{id}/triggers            create trigger
PATCH  /hermes/agent/{id}/triggers/{trigger_id}  update trigger
DELETE /hermes/agent/{id}/triggers/{trigger_id}  delete trigger
GET    /hermes/models                         available models
GET    /hermes/apps/browse                     app store browse
GET    /hermes/apps/content                   app content
GET    /hermes/workspace-store/agents         workspace store
GET    /hermes/workspace-store/agents/search  store search
```

## 4. Data model (query keys + fields)
Query keys (`["workspace-agent", …]`):
```
detail · builder · activity · apps · available-models
schedules · published-versions · connector-readiness · connector-actions
memory · memory-file
```
Fields seen: `agent_id`, `agentName`/`name`, `systemHint`/`system_hint`,
`connectors`/`connector`, `schedule`, `trigger_id`, `persistent_folder`,
`published_version`, visibility.

So an agent = **name + system hints + connectors + a persistent folder + a set
of scheduled triggers + published versions**, surfaced as a chat surface with
activity and an app store.

## 5. Related surfaces
- `hermes/models`, `hermes/apps/browse` — model + app catalogs.
- `workspaceAgents.sidebar.{pin,unpin,unpinFailed}` — sidebar management.
- Connectors come from `/aip/connectors/*` (see `BACKEND-API-MAP.md`).

## 6. Telemetry / flags
- Product events: `CODEX_PLUGIN*`, `CODEX_AGENT_*` families plus
  `agent-activity-item` UI.
- Gate: the `enabled` rule above (account flag) — check via the debug panel's
  `feature-access` section.

## Use it in dev mode
- Query Devtools → filter `workspace-agent` to see cached agent/schedule data.
- Debug panel → `chatgpt-api` to watch live `/hermes/*` requests.
- Statsig map → look for the agent gate.
