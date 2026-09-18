# Hidden debug / internal surfaces in the desktop app

Fresh pass over the app asar. These are not in the UI and not documented.

## 1. Debug Modal — bound to `Ctrl+D`
- Command `toggleDebugModal` (`codex.command.toggleDebugModal`),
  default keybinding **`Ctrl+D`** in the Electron app; available on
  `browser`/`electron`/`extension`.
- Component `DebugModal` (chunk `debug-modal-*.js`) — a resizable side panel
  (`--debug-panel-width`, `debug-panel-selected-section`, "Resize debug panel").

## 2. Debug Settings — hidden toggles
- **GPU Tearing Debug** group: *Disable backdrop blur*, *Disable CSS motion*,
  *Disable scroll fade animation*, *Disable scroll fade mask*,
  *Disable squircles*, *Force opaque web background*.
- **Chrome debug pages** toggle — enables debug-only Chrome pages
  (`chrome://…`) inside the app.
- (chunk `debug-settings-*.js`)

## 3. Debug window + Control window
- `createDebugWindow` → a `Debug` window, 920×840, secondary appearance
  (chunk `debug-window-page-*.js`).
- Command `openControlWindow` → **CmdOrCtrl+Alt+S**; opens a control window that
  speaks the `debug-run-eval-control-*` protocol.
- Debug menu commands (from the main process):
  - *Toggle Query DevTools* — `CmdOrCtrl+Alt+Y`
  - *Toggle React Scan*
  - *Open Deeplink from Clipboard*

## 4. Eval / control harness (in production)
Main↔renderer protocol `debug-run-eval-control-request` /
`debug-run-eval-control-response`, scoped by action. Registered actions:
```
context-action:pick-local-files
context-action:capture-appshot
context-action:record-skill
command:project
command:goal
command:plan-mode
```
The debug-window manager routes these to a primary window and executes them —
i.e. a **client-side test/eval automation channel shipped in the release build**.
Related state messages: `debug-window-client-analytics-history-changed`,
`debug-window-origin-conversation-changed`.

## 5. Full command registry (127 commands)
Exported to `desktop-command-registry.txt`. Notable hidden/less-visible ones:
`openControlWindow`, `toggleDebugModal`, `hotkeyWindow`, `forceReloadSkills`,
`importExternalAgent`, `codexMicroSettings`, `globalDictationHold` /
`globalDictationToggle`, `composer.captureAppshot`, `openAvatarOverlay`
(→ `openPetOverlay`), `switchToMode1/2/3` (chat/work/codex),
`showWorkspaceTabView`, `stepWorkspaceLayout`, `environmentAction1..9`,
`recentThread1..6`, `thread1..9`.

## 6. Full main→renderer message surface (318 types)
Exported to `main-to-renderer-messages.txt`. Covers
`avatar-overlay-*`, `browser-sidebar-*`, `bem-*` (BEM = presentation/artifact
scoring events), `appshot-shortcut`, `ambient-suggestion-generation`,
`codex-micro-device-*`, `debug-*`, `toggle-debug-modal`,
`toggle-query-devtools`, etc.

## 7. Internal route
`/internal/demo-tools` exists in the production router.

## 8. CODename: "sheep"
`x-openai-use-sheep: 1` is sent on the legal API call
(`/legalapi/enabled` via `safeGet`) — an internal feature/versioning flag.

## Why these are "hidden"
None appear in Settings, menus, or docs: a Ctrl+D debug modal, a GPU-tearing
debug group, a chrome:// debug-page toggle, a control window with an eval-action
harness, React Scan / Query DevTools toggles, and an internal demo-tools route —
all shipped enabled in the release bundle.

## Files
- `desktop-command-registry.txt` (127 commands)
- `main-to-renderer-messages.txt` (318 message types)
- de-minified `debug-modal`, `debug-settings`, `debug-window-page` in
  `../analysis/desktop-pretty/` (local)
