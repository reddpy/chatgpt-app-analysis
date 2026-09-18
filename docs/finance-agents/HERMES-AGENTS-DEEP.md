# Hermes / Workspace Agents — deep dive

Follow-up to `HERMES-AGENTS-TRACE.md`, with the detail-page internals, memory
files, and the automation (trigger) model.

## Agent detail page
Tabs (`workspaceAgents.detail.tabs.*`): **Activity · Apps · Memory**
- header: creator attribution, visibility (**private**), `edit`, `view`,
  **Open in ChatGPT** (cross-surface handoff), pin (sidebar).
- `pinError` / `loadError` / "Agent not found" states.

## Memory files
The agent keeps a **persistent memory file system**
(`/hermes/agent/{id}/persistent-folder/tree`, files `.../download_url`).
UI handles: `memory.api`, `memory.chatGpt`, `memory.empty(Description|Title)`,
`memory.fileEmpty`, `fileError`, `fileTooLarge`, `fileTruncated`,
`fileUnsupported`, `readError`. Query keys `["workspace-agent","memory"|"memory-file", …]`.

## Schedules = RRULE automations ("Event subscriptions")
`workspaceAgents.schedules.*`:
- **"Scheduled automation"** editor; **Add automation**; destination **ChatGPT**.
- **RRULE validation** (`invalidRule`): *"Choose a repeating schedule with one
  time per rule, at least 15 minutes apart, without a year, end date, or numbered
  weekday."*
- Preconditions: **publish the agent first** (`errors.unpublished`); **connect/
  reconnect the agent's apps** (`errors.connections`); workspace identity
  (`errors.identityMissing`); permission (`errors.permission`).
- **Slack automations are owner-only** (`errors.slackPermission`).
- Sidebar rows with event subscriptions are labelled **"Event subscriptions"**
  (`sidebarTaskRow.triggers`).

So hermes triggers reuse the same RRULE/heartbeat machinery as the app's
automations, scoped to a published Workspace Agent.

## System hints (capabilities)
Agent prompts reference hints prefixed:
`search`, `picture_v2`, `reason`, `plugin:<slug>`, `connector:<id>`,
`custom_agent:<id>` — i.e. an agent can enable search, image, reasoning,
plugins, connectors, and other custom agents.

## Builder / store / models
- `builder-view` + `detail-view` payloads; `save` / `save_publish`; `clone`;
  `published_versions`.
- `hermes/models` (model catalog), `hermes/apps/browse`, `hermes/apps/content`.
- `hermes/workspace-store/agents` + `/search` — a workspace-level **agent store**.

## Query keys
```
["workspace-agent", { detail, builder, activity, apps, available-models,
                      schedules, published-versions, connector-readiness,
                      connector-actions, memory, memory-file }]
```
Plus `["workspace-agent","connector-actions"|"connector-readiness", …]` and
pins `workspace-agent-pins-by-account-v1`.

## Bottom line
Hermes is a **full agent platform**: author an agent (system hints + connectors +
persistent memory folder + model), publish versions, schedule it via RRULE
automations (ChatGPT or Slack destinations), share it in a workspace store, pin
it to the sidebar, and chat with it — with an activity feed and cross-surface
handoff to ChatGPT.
