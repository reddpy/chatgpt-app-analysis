# The "owl" framework — source provenance

## Verdict

`Codex Framework.framework` is **stock Electron 42.3.0 (Chromium 153.0.8010.36)
with Electron's `shell/` source tree renamed to `owl/`**, plus a bounded set of
OpenAI additions. It is **not** a from-scratch runtime.

Evidence: the binary embeds Chromium `__FILE__` strings; the Electron-layer ones
read `../../owl/browser/api/electron_api_app.cc`, `../../owl/renderer/...`, etc.
That is exactly Electron's `shell/` layout — `shell/browser/api/electron_api_app.cc`
→ `owl/browser/api/electron_api_app.cc`.

## The delta vs Electron v42.3.0

I diffed every embedded `owl/…` path (93 files that contain CHECK/log statements)
against the `shell/` tree of `electron/electron@v42.3.0` (855 files):

| Category | Count | Meaning |
|---|---|---|
| Exact path match | 55 | renamed `shell/` → `owl/`, content ~upstream |
| Relocated (same filename, different dir) | 3 | moved within the fork |
| **No upstream counterpart** | **35** | **OpenAI-authored files** |

`electron-v42.3.0-shell-paths.txt` is the upstream list used for the diff.

### OpenAI-added files (proprietary delta)

```
browser/api/electron_api_autocomplete.cc
browser/api/electron_api_favicon_service.cc
browser/api/electron_api_owl_update_policies.cc      <- explicitly owl-branded
browser/browser_profile_importer_extensions.cc
browser/browser_profile_importer.cc
browser/contents_container_view.h
browser/electron_app_main_process_host.cc
browser/electron_app_window_mac.mm
browser/electron_app_window.cc
browser/electron_dialogs_mac.mm
browser/electron_download_history.cc
browser/electron_popup_menu.cc
browser/electron_session_host.cc
browser/electron_session_top_sites.cc
browser/electron_web_contents_host.cc
browser/electron_web_request_url_loader_factory.cc
browser/electron_web_request_websocket.cc
browser/extension_action_menu.cc
browser/extension_popup.cc
browser/native_glass_controller.cc
browser/native_system_font_catalog_mac.mm
browser/node_app_runner.cc
browser/owl_notification_presenter_bridge_state.cc   <- explicitly owl-branded
browser/pointer_coordinate_safety.cc
browser/system_font_catalog.cc
browser/tracing_coordinator.cc
common/api/electron_api_crashpad_support.cc
common/api/electron_api_shell_platform_mac.mm
common/thread_restrictions.cc
host/content/browser/devtools/child_frame_screenshot_capture.cc
host/content/browser/renderer_host/reparented_surface_capture.cc
host/content/browser/web_contents/inner_web_contents_capture.cc
host/stuck_web_request_reporter.cc
host/third_party/blink/renderer/core/native_glass_style_tracker.cc
host/ui/compositor/native_glass_compositor.cc
```

Notable: the `host/…` files are **patches into Chromium itself** (content/,
third_party/blink, ui/compositor) — screenshots, surface capture, and "native
glass" (macOS translucency). Also present: owl-specific runtime keys
`owl_websocket_interceptor`, `owl_electron_permission_profile_data`,
`owl_native_glass_data_`, `owl_notification_presenter_bridge_state`,
`owl_electron_extension_system_initialized`, `owl_update_policies`.

## What this means for "real source"

- **~74 of the Electron-layer files are public Electron source** (rename only):
  take `electron/shell/<path>` at tag `v42.3.0`.
- **35 files are internal.** No public source exists. Options:
  1. Ghidra the framework binary (270 MB — impractical in full; targeted
     analysis of the `owl_*` code paths is possible but symbol-poor).
  2. Reconstruct behavior from the JS side (`app.asar`), which calls these APIs.
- The framework binary is **not** a source-recovery target the way the Rust was;
  it is 95% Chromium.

## `libaperitif` / `Codex (Aperitif*)` — NOT custom logic

`Libraries/libaperitif.dylib` and the `Codex (Aperitif*).app` helpers are
OpenAI's **rename of Chromium's macOS seatbelt sandbox** components
(`sandbox_apply`, `sandbox_init_with_parameters`, `org.chromium.sandbox`,
`AperitifCheckInitialized`). Their source is Chromium `sandbox/mac/`.
Decompilation is included under `../native-decompiled/aperitif/`.
