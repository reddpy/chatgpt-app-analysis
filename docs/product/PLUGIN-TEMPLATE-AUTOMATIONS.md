# Hermes plugin-template automations

The `pluginTemplateId` on an automation points at a **scheduled task declared by
a plugin**. Schema is in the open-source Rust app-server protocol
(`app-server-protocol/src/protocol/v2/plugin.rs`) — so this is exact.

## How it works
- Plugins are listed via `plugin/list`; each plugin's `PluginDetail` carries:
  ```rust
  pub struct PluginDetail {
      ...
      pub apps: Vec<AppSummary>,
      pub app_templates: Vec<AppTemplateSummary>,
      pub mcp_servers: Vec<String>,
      pub scheduled_tasks: Option<Vec<ScheduledTaskSummary>>,   // ← automations
  }
  ```
- The client flattens `marketplaces[].plugins[].scheduled_tasks` into
  **automation suggestions**, and a chosen one is instantiated as a `cron`
  automation carrying `pluginTemplateId`.
- Automations can also be **reset to their plugin template**
  (`resetAction` / `isUsingDefaults`) — restoring the template's prompt/schedule.

## Explicit "scheduled task" schema
```rust
pub struct ScheduledTaskSummary {
    pub key: String,      // template id within the plugin
    pub name: String,
    pub prompt: String,   // the task prompt
    pub schedule: ScheduledTaskSchedule,
}

pub enum ScheduledTaskSchedule {
    Hourly   { interval_hours: u32, days: Option<Vec<ScheduledTaskWeekday>> },
    Daily    { time: String },
    Weekdays { time: String },
    Weekly   { days: Vec<ScheduledTaskWeekday>, time: String },
}

pub enum ScheduledTaskWeekday { Mo, Tu, We, Th, Fr, Sa, Su }
```
So a plugin declares recurring tasks by **name + prompt + schedule** (hourly with
interval/days, daily, weekdays, or weekly with weekday list + wall-clock time).
`time` is a wall-clock string; the UI's RRULE layer renders it (and enforces
≥15 min spacing, no DTSTART, etc.).

## Related plugin template concept: `AppTemplateSummary`
```rust
pub struct AppTemplateSummary {
    pub template_id: String,
    pub name: String,
    pub description: Option<String>,
    pub category: Option<String>,
    pub canonical_connector_id: Option<String>,
    pub logo_url: Option<String>,
    pub logo_url_dark: Option<String>,
    pub materialized_app_ids: Vec<String>,
    pub reason: Option<AppTemplateUnavailableReason>, // NotConfiguredForWorkspace | NoActiveWorkspace
}
```
App templates get **materialized** into apps (connector-backed). Unavailable when
the workspace isn't configured / has no active workspace.

## Plugin catalog / marketplace model (same file)
- `PluginSource`: `local{path}` | `git{url,path,ref,sha}` | `npm{package,version,registry}` | `remote`.
- `PluginListMarketplaceKind`: `local`, `vertical`, `workspace-directory`,
  `shared-with-me`, `created-by-me-remote`.
- Install policy: `NOT_AVAILABLE | AVAILABLE | INSTALLED_BY_DEFAULT`
  (+ source `WORKSPACE_SETTING | IMPLICIT_CANONICAL_APP`), `PluginAuthPolicy`
  `ON_INSTALL | ON_USE`, `PluginAvailability` `AVAILABLE | DISABLED_BY_ADMIN`,
  `PluginDisabledReason` `disabled_by_admin | plan_not_eligible | required_app_unavailable`.
- **Sharing**: `PluginShareSave/UpdateTargets`, discoverability
  `LISTED | UNLISTED | PRIVATE`, targets with roles `reader | editor | owner`,
  principals `user | group | workspace`, `can_publish_to_workspace`, `share_url`.
- `PluginInterface.default_prompt` — up to 3 starter prompts (≤128 chars each).
- `hooks` per plugin (`HookHandlerMetadata`: command / mcpTool / prompt / agent).

## Client-side flow (JS)
- `plugin/list` → `marketplaces.flatMap(e => e.plugins.flatMap(p => p.templates…))`.
- `Mn({pluginId, template})` builds a selection `{ pluginTemplateId, name,
  description: template.prompt, icon, scheduleConfig }`.
- `pluginTemplateId` decodes back to a pluginId (`ln()`), and the automation
  editor:
  - `kind === "cron"` + `pluginTemplateId` → load plugin template.
  - shows `pluginTemplatePreview` / `pluginTemplateGroups`,
  - offers "reset to plugin template" (defaults) and "run now".

## Why it's juicy
It means **any plugin (marketplace, workspace, git, npm, or remote) can ship
recurring agent tasks** — a named prompt + a schedule (hourly/daily/weekdays/
weekly) — that users can adopt as **cron automations** in one click, and reset.
Combined with Hermes agents, the app supports *third-party-scheduled autonomous
tasks*.
