# owl vs stock Electron 42.3.0 — binary diff

Compared:
- **stock**: `electron-v42.3.0-darwin-arm64` → `Electron Framework` (182 MB,
  `com.github.Electron.framework`, 12,599 symtab symbols)
- **owl**: `Codex Framework` (258 MB, `com.openai.codex.framework`, 153.0.8010.36,
  14,292 symtab symbols)

## Headline: owl is **not** a renamed Electron

Stock Electron embeds **46** `../../chrome/` source paths. owl embeds **2,044**.
owl adds **136 Chrome `chrome/browser/…` subdirectories** that stock Electron does
not have. So owl = **Electron API shell + a full Chrome browser layer**.

- owl renames Electron's `electron/shell/` tree to `owl/` (93 paths) and adds 35
  files → the Electron-compatible API layer.
- On top of that it compiles the **Chrome browser** (tabs, history, downloads,
  bookmarks, autofill, sign-in, sync, enterprise, payments, safe browsing,
  WebAuthn, supervised users).

## owl-only Chrome subsystems (absent in stock Electron)

AI / agentic:
```
chrome/browser/actor/                 # Chrome's browser-agent framework (85 files)
chrome/browser/glic/                  # Gemini-in-Chrome (75 files)
chrome/browser/ai/
chrome/browser/optimization_guide/    # on-device model management
chrome/browser/accessibility_annotator/
chrome/browser/screen_ai/
chrome/browser/compose/
chrome/browser/contextual_tasks/
chrome/browser/contextual_cueing/
chrome/browser/dictation/
chrome/browser/segmentation_platform/
```
Browser: `autofill/`, `autocomplete/`, `bookmarks/`, `download/`, `enterprise/`,
`payments/`, `signin/`, `sync/`, `safe_browsing/`, `webauthn/`,
`supervised_user/`, `data_sharing/`, `device_identity/`, `digital_credentials/`,
`gcm/`, `performance_manager/`.

### `chrome/browser/actor/` — the interesting one
Chrome's built-in browser agent:
```
actor_keyed_service.cc, actor_task.cc, actor_metrics.cc, execution_engine.cc,
tab_observation_controller.cc
tools/attempt_login_tool.cc, tools/attempt_otp_filling_tool.cc,
tools/history_tool.cc, tools/load_and_extract_content_tool.cc,
tools/page_tool.cc, tools/script_tool_host.cc, tools/wait_tool.cc,
tools/tool_controller.cc, tools/observation_delay_controller.cc
ui/actor_overlay_web_view.cc, ui/actor_ui_tab_controller.cc,
ui/actor_ui_window_controller.cc, ui/event_dispatcher.cc,
ui/handoff_button_controller.cc, ui/task_list_bubble/actor_task_list_bubble_controller.cc
```
(This is Chrome's own agentic-browsing stack — login/OTP tools, observation
loop, overlay UI, task list, handoff — compiled straight into owl.)

## Frameworks / Libraries diff

| | stock | owl |
|---|---|---|
| Graphics | `libEGL.dylib`, `libGLESv2.dylib`, `libffmpeg.dylib`, `libvk_swiftshader` | `libvulkan.dylib`, `libvk_swiftshader` |
| Extra | — | `IwaKeyDistribution`, `MEIPreload`, `PrivacySandboxAttestationsPreloaded`, **`libaperitif.dylib`** |

owl swaps EGL/GLES for **Vulkan**.

## macOS "Liquid Glass"

owl-only symbols/strings: `NSGlassEffectView`, `NSGlassEffectContainerView`,
`glassEffect`, `glassEffectClassic`, `glassEffectModern`, `showGlassEffectEnabled`
→ owl uses Apple's new **glass** APIs (macOS 26) for native chrome.

## Symbols
owl has **2,259** symbols stock lacks (1,323 net after removals); includes the
GlassEffect classes and the `chrome/actor`, `glic`, `compose`, `autofill`,
`dictation` code.

## Verdict
`owl` is a **Chromium/Chrome build with the Electron shell renamed to `owl/`**,
i.e. **Electron + Chrome + Chrome's AI subsystems (actor, glic/Gemini,
optimization-guide, compose, accessibility-annotator, screen-ai)**. That's why
the framework is 258 MB vs 182 MB, why it has `OwlHistoryOverlayView`,
`OwlSiteSettingsEmbed`, `OwlRemoteSearchSuggestions`, tray/dock menus — and why
`chrome/browser/actor/` (Chrome's browser agent) is present in OpenAI's desktop
app.

Practical consequence: much of owl's extra surface is **upstream Chrome code**
(public), so the proprietary delta is smaller than it looked: the `owl/` Electron
shell (93 paths, 35 added files) + OpenAI's integration/UI + the `host/` Chromium
patches we found earlier.
