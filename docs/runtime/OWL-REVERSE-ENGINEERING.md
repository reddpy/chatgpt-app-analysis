# Reverse-engineering owl — what's possible

## Verdict
Full decompilation of `Codex Framework` is infeasible (258 MB binary, 209 MB of
`__text`, ~55 M instructions, 16 GB RAM). But **targeted** RE works well, and we
recovered owl's custom API implementations.

## Method (works)
1. Extract embedded source paths (`../../owl/...`) — reveals the fork's layout.
2. Extract owl-specific string constants (API names, feature keys, markers) and
   their VM addresses.
3. **Custom ARM64 xref scanner**: decode every `ADRP`+`ADD` pair in `__text`
   with numpy to find code referencing those strings (Capstone/r2 too slow).
4. Recover function starts via a `RET`-boundary scan.
5. **Targeted Ghidra** import with auto-analysis disabled; disassemble +
   decompile only those functions.

This avoids the OOM/hours-long full analysis while still yielding the owl code.

## What we recovered
- **14 owl-specific API functions** decompiled (this pass), plus **209 functions**
  referencing owl strings (earlier pass) → `Codex Framework.owl.c`.
- Recovered owl API surface (bound to JS):
  `owlUpdatePolicies`, `owlFeatures`/`isOwlFeatureEnabled`/`OwlFeatureEnabled`,
  `owlNativeGlass{Enabled,Container,Spacing,Material}`, `owlBrowserCrashCounter`,
  `owlSessionInitialized`, `owl_electron_extension_system_initialized`,
  `owl_electron_permission_profile_data`, `owl_websocket_interceptor`,
  `owl_notification_presenter_bridge_state`.
- owl native UI classes: `OwlNativeGlass`, `OwlNativeCaret`,
  `OwlHistoryOverlayView`/`SwipeOverlayView`, `OwlSiteSettingsEmbed`,
  `OwlRemoteSearchSuggestions`, `OwlElectron{Tray,Dock,Application}MenuController`,
  `OwlElectronProgressBar`, `OwlExtensionDialog`, `OwlDragFile/LinkSource`.

## Hidden mechanisms found this pass

### owl `owlUpdatePolicies` — forced-update / relaunch policy
The app reads `require('electron').owlUpdatePolicies` and subscribes to
`relaunch-notification-policy-changed`. Decoded policy shape:
```
relaunchNotification          // integer; 2 => forced install
relaunchNotificationPeriodMs
relaunchWindow                // { start: {hour,minute}, duration_mins }
relaunchFastIfOutdatedDays
```
Behavior in the app: `scheduleForcedUpdateInstall()` → when
`relaunchNotification === 2`, it waits until inside `relaunchWindow` then calls
`installForcedUpdate()`, retrying. So owl enables **MDM/enterprise-controlled
forced updates + relaunch windows**.

### owl runtime feature gates
`isOwlFeatureEnabled(name)` → `window.electron.app.isRuntimeFeatureEnabled(name)`
(throws `Unsupported Owl feature ...` for unknown names). Feature set is
**server-provided** — `~/Library/Application Support/Codex/owl-feature-bootstrap-cache.json`
currently: enabled `[OwlHistory, OwlPrinting]`, disabled
`[OwlExtensions, OwlOpenAIGoLinks]`.

### Other owl internals
- `owl_websocket_interceptor` — owl intercepts WebSocket traffic.
- `owlNativeGlass*` — native macOS glass/material (translucency) with
  `misc_data_->owl_native_glass_data_`, container/spacing/material/hidden knobs.
- `owl_electron_extension_system_initialized`,
  `owl_electron_permission_profile_data` — extension system + permission profile.
- `owl_notification_presenter_bridge_state` — a notification presenter bridge.
- `owlBrowserCrashCounter` — renderer crash telemetry exposed as a window log field.

## Limits & ways to go further
- Decompiled output is low-level Chromium C++ (optimized, no symbols) — readable
  structure, not clean source.
- **Diff vs stock Electron 42.3.0** is the definitive next step to isolate owl's
  exact patch set (download Electron 42.3.0 darwin-arm64, compare the embedded
  path lists + symbol/string sets). We already have the path-level diff (55
  shared, 3 relocated, 35 added).
- Bigger region decompilation needs more RAM (32–64 GB) or a cloud instance.
- Contracts can often be reconstructed from the app JS (as done for
  `owlUpdatePolicies`).

## Artifacts
- `../native-decompiled/owl-framework/Codex Framework.owl.c` (209 functions)
- `owl-api/Codex Framework.owl.c` (14 owl-API functions, this pass)
- `owl-api-func-starts.txt`, `owl-api-xrefs.txt`, `xref_owlapi.py`
