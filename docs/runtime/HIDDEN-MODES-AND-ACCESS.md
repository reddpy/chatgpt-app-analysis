# Hidden modes & how to reach them

## 1. The master gate: build flavor

Every debug/internal surface is gated by the app's **build flavor**:

```js
isInternal(flavor)      // dev | agent | nightly | internal-alpha
allowDebugMenu(flavor)  === isInternal(flavor)
allowDevtools(flavor)   === isInternal(flavor)
supportsReactScan(flavor)  // dev | agent | …
enablesWindowsSandboxService(flavor)  // nightly | internal-alpha
```

Resolution order (verified in code):
```js
resolve(){ let e = process.env.BUILD_FLAVOR;          // 1. env WINS
           let t = parse(e);
           return t ? t : readFromPackageMetadata()    // 2. package.json codexBuildFlavor
                      || (NODE_ENV==='production' ? 'prod' : 'dev'); }
```

The shipped app is **`prod`** (`package.json` `codexBuildFlavor: "prod"`). Valid
flavors: `dev`, `agent`, `nightly`, `internal-alpha`, `public-beta`, `prod`.

### Access
Launch the binary with the env var set (env takes precedence over metadata):

```sh
BUILD_FLAVOR=internal-alpha /Applications/ChatGPT.app/Contents/MacOS/ChatGPT
# or: dev | agent | nightly
```

## 2. What that unlocks
- **Debug menu** (`allowDebugMenu`) and **devtools** (`allowDevtools`).
- **Debug Modal** via **Ctrl+D** (`toggleDebugModal`).
- **React Scan** toggle and **Query DevTools** (CmdOrCtrl+Alt+Y).
- **Control window** (`openControlWindow`, CmdOrCtrl+Alt+S) with the
  `debug-run-eval-control` action harness.
- **Debug Chrome pages** toggle → enables `chrome://…` pages.
- Internal-only feature surfaces (gated by `isInternal`): **notebook**,
  **pull-requests** (code-review plugin), **deep research**.
- `enablesWindowsSandboxService` (nightly/internal-alpha).

Note: non-prod flavors also switch the SQLite DB to `codex-dev.db` and (for
dev/agent) the bundle id to `com.openai.codex.dev` / `.agent`.

## 3. Hidden settings keys
App-settings schema (snake_case) includes flags never shown in the UI:
```
developer_mode
dictation_enabled
enable_device_code_auth
enable_flora_network_access
connector_search_enabled
connector_enforce_csp_in_dev_mode
default_to_study_mode
contrast_mode
configurable_thinking_effort
bazaar_personalization_enabled
disabled_by_admin
```
Global-state flags (`~/.codex/.codex-global-state.json`):
```
electron-internal-update-cdn-enabled
electron-desktop-otraces-sample-rate
electron-openai-mcp-form-elicitations-enabled
electron-avatar-overlay-open
electron-avatar-overlay-bounds
electron-initial-follow-up-queue-mode
electron-local-remote-control-installation-id
electron-remote-hosted-pip-task-visibility-state
electron-persisted-atom-state
```
The debug Chrome pages flag `DEBUG_CHROME_PAGES_ENABLED` is stored here, and is
coerced to `false` unless `allowDebugMenu(resolve())` (i.e. needs an internal
build flavor).

## 4. Config surface (`~/.codex/config.toml`)
Already-used sections (from the live config on this machine):
```
[desktop]              conversationDetailMode, dock-icon-preference,
                       followUpQueueMode, appearanceTheme
[desktop.open-in-target-preferences]   global, perPath
[desktop.appearanceLightChromeTheme]   accent, contrast, ink, surface, opaqueWindows, fonts, semanticColors
[features]             js_repl
[marketplaces.openai-bundled] / [marketplaces.openai-primary-runtime]
[mcp_servers.node_repl]        env: BROWSER_USE_AVAILABLE_BACKENDS,
                                      BROWSER_USE_TINYSKY_ENABLED, …
[mcp_servers.computer-use]
[mcp_servers.<user>]
```

## 5. Modes you can enter
- **App modes**: `switchToMode1/2/3` → chat / work / codex.
- **Plan mode**: `composer.togglePlanMode`; **Fast mode**:
  `composer.toggleFastMode`; **Worktree mode**: `composer.toggleWorktreeMode`;
  **Work run location**: `composer.toggleWorkRunLocation`.
- **Temporary chat**: `temporaryChat`; **Side chat**: `openSideChat`.
- **Hotkey window**: `hotkeyWindow`.
- **Realtime voice / Dictation**: `realtimeVoice`, `globalDictationToggle/Hold`.
- **Pet overlay**: `openAvatarOverlay` (→ openPetOverlay).
- **Internal**: `/internal/demo-tools`; debug modal (Ctrl+D).

## 6. State/settings files
- `~/.codex/config.toml` — agent + `[desktop]` config
- `~/.codex/.codex-global-state.json` — UI/global state flags (above)
- `~/Library/Application Support/Codex/` — Chromium profile, `owl-feature-bootstrap-cache.json`, `statsig-state.json`
- `~/Library/Application Support/Codex/browser-sidebar-page-states.json`

## VERIFIED — actually done on this machine

Steps that worked:
1. Stub the bundled Git the internal build expects (production omits it):
   ```sh
   R=/Applications/ChatGPT.app/Contents/Resources
   mkdir -p "$R/git/bin" "$R/git/etc" "$R/git/libexec" "$R/git/share/git-core"
   ln -sfn /usr/bin/git "$R/git/bin/git"; : > "$R/git/etc/gitconfig"
   ln -sfn /Applications/Xcode.app/Contents/Developer/usr/libexec/git-core "$R/git/libexec/git-core"
   ln -sfn /Library/Developer/CommandLineTools/usr/share/git-core/templates "$R/git/share/git-core/templates"
   ```
2. `BUILD_FLAVOR=internal-alpha /Applications/ChatGPT.app/Contents/MacOS/ChatGPT`

Observed evidence:
- `Launching app ... allowDebugMenu=true allowDevtools=true buildFlavor=internal-alpha`
- app-server connected (`codex 0.155.0-alpha.2.6`), main window loaded.
- **Press Ctrl+D → Debug Modal opened**; persisted
  `electron-persisted-atom-state.debug-panel-selected-section` = `chatgpt-api`
  (also seen: `product-events`). Sections include `threads`, `chatgpt-api`,
  `product-events`.
- Hidden global hotkey registered at launch: **appshot = DoubleCommand**
  (`Registering appshot hotkey hotkey=DoubleCommand`).

Notes:
- The `(Alpha)` userData path was ignored; it used the real profile
  `~/Library/Application Support/Codex`.
- Stubbing `Resources/git` **breaks the app's code signature** (Sealed Resources).
  Remove the `Resources/git` dir to undo.
- The running instance can be closed with `pkill -f MacOS/ChatGPT`.

## Bottom line
Yes — the debug/internal surface is **reachable** and gated by a single runtime
switch: **`BUILD_FLAVOR`** (`dev`/`agent`/`nightly`/`internal-alpha`). Setting it
turns on the debug menu, Ctrl+D debug modal, devtools, React Scan, control-window
eval harness, `chrome://` debug pages, and internal-only features. Hidden settings
(`developer_mode`, etc.) and global-state flags live in the files above.
