# owl native → JavaScript mapping

Connects owl's native code to the JS API surface the app actually calls.
Method: diffed owl vs stock Electron native modules; extracted `_linkedBinding`
names and JS APIs from owl's built-in JS + the app asar; found the native
implementation functions via ARM64 string-xref scanning + targeted Ghidra.

## owl's only JS-visible native module addition

Owl and stock Electron register the same 49 `electron_*` native modules, with two
differences:

| | module |
|---|---|
| **owl-only** | `electron_browser_owl_update_policies` |
| stock-only | `electron_browser_client`, `electron_browser_main_parts`, `electron_browser_main_parts_posix`, `electron_common_features` |

owl **replaced `electron_common_features`** with its own runtime-feature system.

## The mapping table

### 1. Forced-update / relaunch policy
| Layer | Value |
|---|---|
| JS binding | `process._linkedBinding('electron_browser_owl_update_policies').owlUpdatePolicies` |
| JS API | `getRelaunchNotificationPolicy()`, `.on('relaunch-notification-policy-changed', …)` |
| Native binding | `electron_browser_owl_update_policies` |
| Native function | `FUN_047f3694` (refs `owlUpdatePolicies` @ `0xebd001c`) |
| owl source | `owl/browser/api/electron_api_owl_update_policies.cc` |
| App usage | `observeRelaunchNotificationPolicy()` → `scheduleForcedUpdateInstall()` → `installForcedUpdate()` |
| Policy shape | `relaunchNotification` (2 = force), `relaunchNotificationPeriodMs`, `relaunchWindow {start{h,m}, duration_mins}`, `relaunchFastIfOutdatedDays` |

### 2. Runtime feature gates (replaces `electron_common_features`)
| Layer | Value |
|---|---|
| JS API | `app.isRuntimeFeatureEnabled(name)` |
| App wrapper | `isOwlFeatureEnabled(name)` (returns false on `Unsupported Owl feature: …`) |
| Native | `electron_browser_app` (method `isRuntimeFeatureEnabled`) |
| Server source | `owl-feature-bootstrap-cache.json` → `enabledOwlFeatureNames` / `disabledOwlFeatureNames` |

### 3. Native glass (macOS Liquid Glass)
| Layer | Value |
|---|---|
| JS bundle | `electron/js2c/native_glass_renderer` → `installNativeGlassRenderer()` |
| JS surface | `nativeGlass.onSupportChange(cb)`, `nativeGlass.fallbackReady()`, sets `data-owl-native-glass="active|fallback"` |
| Native bindings | `owlNativeGlassEnabled`, `owlNativeGlassContainer`, `owlNativeGlassSpacing`, `owlNativeGlassMaterial` |
| Native classes | `OwlNativeGlass`, `OwlNativeGlassData`, `OwlNativeGlassMaskGroupView` |
| owl source | `owl/browser/native_glass_controller.cc` (23 fns), `owl/host/third_party/blink/renderer/core/native_glass_style_tracker.cc`, `owl/host/ui/compositor/native_glass_compositor.cc` |
| macOS API | `NSGlassEffectView`, `NSGlassEffectContainerView`, `glassEffect{Classic,Modern}` |
| Native state | `misc_data_->owl_native_glass_data_` |

### 4. WebContents adoption / tab transfer (the big JS patch)
| Layer | Value |
|---|---|
| JS API | `webview.adoptWebContents(id)`, events `-webview-transferred` / `web-contents-transferred` |
| JS attrs | `owl-webcontents-adoption-lease`, `owl-webcontents-adopted-web-contents-id`, `webviewrole="tab"` |
| IPC channels | `GUEST_VIEW_MANAGER_TRANSFER_GUEST`, `GUEST_VIEW_MANAGER_ATTACH_GUEST`, `GUEST_VIEW_MANAGER_DESTROY_GUEST` |
| Native bindings | `electron_browser_web_contents`, `electron_browser_web_view_manager` |
| Native classes | `OwlBlockWebViewFrameAttachmentForTesting`, `OwlRenderWidgetHostViewMacDelegate` |
| owl sources | `owl/browser/electron_web_contents_host.cc` (6 fns), `owl/browser/electron_popup_menu.cc` (10), `owl/browser/electron_app_window_mac.mm`, `owl/browser/electron_app_window.cc` |
| App usage | Browser tab transfer ("Browser transfer receiver is unavailable" / `adoptWebContents`) |

### 5. Other owl sources found in the native decompile
| owl file | # fns | JS-facing meaning |
|---|---|---|
| `owl/browser/native_glass_controller.cc` | 23 | glass (above) |
| `owl/browser/electron_popup_menu.cc` | 10 | native popup menu |
| `owl/browser/node_app_runner.cc` | 9 | runs bundled Node app |
| `owl/browser/electron_web_contents_host.cc` | 6 | WebContents host (tabs) |
| `owl/browser/electron_app_window_mac.mm` / `electron_app_window.cc` | 3 | window shell |
| `owl/browser/electron_dialogs_mac.mm` | 3 | native dialogs |
| `owl/browser/owl_notification_presenter_bridge_state.cc` | 2 | notification bridge state |
| `owl/browser/electron_download_history.cc` | 2 | download history |
| `owl/browser/electron_app_main_process_host.cc` | 2 | main-process host |
| `owl/host/stuck_web_request_reporter.cc` | 2 | stuck-request reporting (Chromium patch) |
| `owl/host/content/.../child_frame_screenshot_capture.cc` | 1 | screenshot capture (Chromium patch) |
| `owl/host/content/.../reparented_surface_capture.cc` | 1 | surface capture (Chromium patch) |
| `owl/browser/browser.cc`, `browser_mac.mm`, `accelerator_util.cc`, `electron_session_host.cc`, `electron_url_loader_factory.cc`, `electron_web_request_url_loader_factory.cc`, `browser_profile_importer_extensions.cc` | 1 each | browser-layer integration |

## How to use this
- Native side: `owl-native-code/Codex Framework.owl.c` (2,854 functions) +
  `owl-file-to-funcs.txt` (file → function addresses).
- JS side: `../feature-code/` and `../js-prompts/` (de-minified app JS) and
  `owl-vs-electron/` (built-in JS bundles + `native_glass_renderer.js`).
- Full native module list: `/tmp`-style diff is captured in this doc; owl-only =
  `electron_browser_owl_update_policies`.

## Bottom line
owl's JS-facing native surface is tiny and precisely identifiable: **one added
native module** (`owl_update_policies`), **one replaced module**
(`electron_common_features` → owl runtime features), and two JS/native feature
concepts — **native glass** and **WebContents adoption/transfer** — plus the
upstream Electron modules. Everything else the app uses is stock Electron.
