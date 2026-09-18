# Desktop app — hidden/unreleased feature inventory

Derived from the shipped Electron bundle: 8,232 unique feature/chunk names
(`desktop-asset-feature-map.txt`), the client RPC verbs
(`desktop-hidden-features.txt`), locale loaders, and strings.

The important framing: **almost all of this is already in the bundle you have** —
it's just not exposed in the UI (feature-flagged, entitlement-gated, or
server-gated). So "extracting" it means inventorying, not hacking.

## Product surfaces visible in the bundle

### App generation ("appgen") — build & publish apps/sites
`appgen-page`, `appgen-database-page`, `appgen-project-row`, `appgen-settings-page`,
`appgen-share-dialog`, `appgen-change-slug-dialog`, `appgen-publication-terms-route`,
`appgen-site-card`, `appgen-site-detail-route`, `appgen-access`,
`appgen-access-state-messages`, `appgen-disabled-tooltip`, plus `site-preview`,
`site-details`, `sites-default-thumbnail`. A full "generate an app with a DB and
publish it at a URL" product.

### Pull requests / code review
`pull-request-*` (route, actions, checks-summary, code-review, code-review-navigation,
comment, detail-model/query, revision-queries, revision-tab, status-label,
readonly-comment, fix-automation, fix-workflow, error-description),
`open-thread-pull-request-{code-tab,side-panel-tab}`, `git-settings`,
`code-review-settings`, `review-file-source-tab-sync`. Native PR review inside the app.

### Library (files)
`library-page`, `library-trash-analytics`, `library-file-preview-kind`,
`library-cloud-file-preview`, `library-move-dialog`, `library-create-dialog`,
`library-empty-trash`, `library-breadcrumbs`, `library-item-actions/dialogs`.

### Automations + Inbox + Tasks
`automations-page`, `automation-dialog`, `automation-frequency-section`,
`automation-side-panel-tab`, `automation-delete-confirmation-dialog`,
`open-automation-side-panel-tab`, `inbox-*`, `home-task-suggestions`,
`inbox-automation-runs-mark-all-read`, `heartbeat-automation-thread-state-changed`,
`task-announcement-candidates`, `chatgpt_tasks`.

### Skills + Plugins + MCP
`skills-page`, `skills-page-grid`, `skills-settings`, `skill-data`, `skill-details`,
`skill-exploration-labels`, `chatgpt-skill-source-dialog`,
`recommended-skill-statsig-overrides`; `plugins-store-page`, `plugins-page`,
`plugin-detail-page`, `plugin-installation-content`, `plugin-picker-menu-content`,
`plugin-mcp-app-deep-link-page`, `plugin-connected-account-links`,
`plugin-disabled-reason`; `mcp-settings`, `mcp-extension-*`, `mcp-app-*`,
`mcp-server-elicitation-request-panel`, `mcp-tool-approval-message`.

### Multi-agent (subagents)
`chatgpt-subagents-panel`, `subagent-panel`, `subagent-row`, `subagents`,
`local-conversation-subagents-panel-tab`, `chatgpt-subagent-final-response-query`,
flag `enable_fanout`.

### Realtime voice
`realtime-voice-launch-surface`, `realtime-voice-home-announcement`,
`realtime-voice-handoff-target`, `realtime-buffered-audio-worklet`,
`realtime-voice-{mute,unmute}.wav`, `realtime-{start,end}.wav`, `chatgpt-voices-query`.

### Appshots (screen capture → content)
`appshots-settings`, `appshot-demo.mp4`, `appshot-logo`, flag `appshots_enabled`,
`browser-use-session-route-capture`.

### Chronicle (the closed Rust binary, surfaced in UI)
`chronicle-settings-page`, flag `skysight_gate_enabled`. Screen-recording → memory
pipeline has a settings page — so the one closed Rust component is user-visible.

### Codex Micro (hardware)
`codex-micro-settings`, `codex-micro-onboarding-animation`,
`codex-micro-onboarding-host`, `codex-micro-analog-action-title`.

### Avatar overlay (companion/mascot)
`avatar-overlay-native-page`, `avatar-overlay-quick-chat-bar`,
`avatar-overlay-debug-state`, `avatar-mascot-button`, `avatar-artwork`,
`electron-avatar-overlay-*` RPCs.

### Design editor (live-page editing/annotation)
`design-editor-state`, `browser-sidebar-design-overlay-*`,
`browser-sidebar-comment-overlay-design-scrub-changed`,
`browser-sidebar-tweaks-enabled-changed`, `set-design-modifier-pressed`,
`design-review.pptx`, `compose-canvas-edit-star.svg`.

### Remote / Cloud / Environments
`remote-connections-page`, `remote-connection-editor-draft`,
`remote-workspace-root-dialog`, `remote-conversation-page`, `remote-middleware`,
`remote-hosted-pip-*`; `cloud-environments-settings-page`, `local-environments-settings-page`,
`cloud-browser-preview`, `cloud-browser-side-panel`, flag `local_remote_control_enabled`.

### Worktrees / Git
`pending-worktree-{create,cancel,continue}` RPCs, `git-settings`, `folder-sync`.

### Billing / plans / credits
`billing-settings`, `billing-analytics`, `billing-state`, `plan-summary-page`,
`read-saved-plan-content`, `editable-plan-tab`, `subscription-update-plan`,
`is-plan-event-enabled`, `invoice`, `credit-card-*`, `budget-planner.xlsx`.

### Security / safety / enterprise
`security-route`, `security-settings`, `security-keys`, `safety-settings`,
`business-switch-workspace-dialog`, flags `admin_work_mode_enabled`,
`admin_work_local_enabled`, `member_profiles_enabled`, `destructive_enabled`,
`disable_auto_review`, `disable_diffing`, `restore_disabled`.

### Misc surfaces
`pets-settings-route` (the undocumented "pets" endpoint has a settings page),
`global-dictation-page`, `hotkey-window-{new-thread,thread}-page`,
`codex-thread-report-dialog`, `thread-usage-breakdown`, `import-settings`,
`connector-settings-redirect`, `pca-connector-*`, plus settings pages for
`general, appearance, notifications, keyboard-shortcuts, personalization,
analytics, storage, debug, computer-use, browser-use, local-remote`.

## Feature-flag tokens found in the main process

```
admin_work_local_enabled        admin_work_mode_enabled
member_profiles_enabled         open_world_enabled
skysight_gate_enabled           full_cdp_access_enabled
webmcp_enabled                  local_remote_control_enabled
enable_fanout                   enable_account_switching
destructive_enabled             computer_history_{enabled,disabled}
bundled_plugin_enabled          plugins_disabled
inactive_thread_inactivity_tracking_enabled
primary_runtime_auto_update_disabled
browser_session_lookup_disabled remote_connection_retry_disabled
restore_disabled                disable_auto_review
disable_diffing                 enable_plugin
```

## owl framework native features (binary class names)

`OwlNativeGlass`, `OwlNativeCaret`, `OwlHistoryOverlayView`,
`OwlHistorySwipeOverlayView`, `OwlSiteSettingsEmbed`, `OwlRemoteSearchSuggestions`,
`OwlElectron{TrayMenu,DockMenu,ApplicationMenu}Controller`,
`OwlElectronProgressBar`, `OwlExtensionDialog`, `OwlDragFileSource`,
`OwlDragLinkSource`, `OwlUpdatePolicies`, `Owl{openAIGoLinks}`.
Server-gated at runtime — current state on this machine:
`enabled: [OwlHistory, OwlPrinting]`, `disabled: [OwlExtensions, OwlOpenAIGoLinks]`.

## Can we "extract all these"?

Yes, in the sense that matters:
- **Names/surfaces** — fully enumerated here (`desktop-asset-feature-map.txt`, 8,232).
- **Code** — present in `../asar-extract/webview/assets/<feature>-<hash>.js`; we can
  prettify/read any of them on request.
- **What gates them** — feature flags (above) + server Statsig gates + entitlements;
  so *enabling* them is a policy/server matter, not a code-extraction matter.
- **What we can't get** — the server-side implementations they call (e.g. appgen
  build pipeline, PR backend, cloud environments).
