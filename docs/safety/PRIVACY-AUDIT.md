# Privacy audit — ChatGPT/Codex desktop app

Scope: the running application (Electron shell, owl framework, native addons,
bundled plugins, `codex` sidecar). Evidence: recovered JS, binary strings,
decompiled native addons, and plugin docs.

## 1. Is it keylogging? — No general keylogging; global modifier monitor only

- `native/bare-modifier-monitor` uses **`addGlobalMonitorForEventsMatchingMask:`**
  (a global NSEvent monitor) but the symbols are all modifier-scoped:
  `NSEventModifierFlags`, `DoubleModifierMonitorSession`,
  `ReleaseModifierMonitor`, `MonitorMode`. It watches **modifier keys**
  (for double-tap-modifier triggers), not character input.
- `sky.node` also references `addGlobalMonitorForEventsMatchingMask:handler:`.
- The app ships `input-monitoring-permission.node` to check the TCC
  **Input Monitoring** permission — i.e., it does request that permission (for
  the modifier monitor).
- **Verdict:** global *modifier* monitoring, no ambient keystroke capture found.

## 2. Typed / selected text IS recorded — by Computer History

`computer-history` plugin (the "chronicle"/`skysight` pipeline) records a
**rolling local event stream** containing, per its own docs:
> app names, window titles, **URLs, selected text, typed text**, and timestamps

- Stored locally: `<eventStreamRootPath>/segments/<ts>/events.jsonl` + `metadata.json`.
- Controls: `paused` / `stopped`; per-app and per-URL `observe` /
  `do_not_observe`; **private browsing is always excluded**.
- It's opt-in, local, but **agent-readable** (the skill says to `rg` the raw JSONL)
  and feeds "memories".
- **`record-and-replay`** additionally records user actions (clicks, typing,
  window content) with an explicit pre-capture confirmation.

This is the single biggest privacy surface: continuous typed/selected text.

## 3. Audio — mic + system audio capture, claim "never saved"

- `native/system-audio-spectrum` captures **system audio** (CoreAudio, TCC
  `kTCCServiceAudioCapture`) to drive a UI animation. Its usage string says:
  *"Audio is processed locally and never saved."*
  - Decompiled code shows **no network calls and no file writes** (only an
    `NSFileHandle` import, consistent with CoreAudio). The local-only claim
    holds in the driver.
  - It is still a system-audio capture surface (entitlement
    `com.apple.security.device.audio-input`, `NSAudioCaptureUsageDescription`).
- Microphone: used by realtime voice / dictation (`global-dictation-page`),
  audio input entitlement, `SKY_ENABLE_AUDIO` / `NODE_REPL_ENABLE_AUDIO` flags.
- No hidden background recording of the mic found (voice is user-initiated).

## 4. Screen & accessibility capture (agent-mediated)

- **Screenshots**: computer-use, appshots (`appshots-settings`), browser
  `TabScreenshot`, `TabPageAssets*`. Sent to vision models.
- **Screen recording → memories**: `codex_chronicle` (closed binary,
  ScreenCaptureKit) + `chronicle-settings-page`.
- **Accessibility trees**: computer-use reads the AX tree of apps
  (`TabAxGetState`, `TabAxAction`); record-and-replay diffs AX.
- **Clipboard**: agent-invoked (`TabClipboardRead/Write`, `clipboard.read`
  ×12) — not ambient polling.

## 5. Telemetry egress (what leaves the machine)

| Sink | Endpoint | Notes |
|---|---|---|
| Sentry | `o33249.ingest.us.sentry.io` | errors + breadcrumbs; breadcrumbs truncated to **2 KiB**, ~20 kept; `attachStacktrace` |
| Statsig | `ab.chatgpt.com/v1`, `statsigcdn.openai.com` | feature gates + custom events |
| Segment/CES | `/ces/v1/rgstr`, `/v1/log_event`, `/ces/v1/telemetry` | product analytics |
| Datadog RUM | app proxy `/telemetry/intake` (clientToken) | **respects `navigator.doNotTrack !== '1'`** |
| OpenTelemetry | OTLP exporter | Rust `codex.*` spans/metrics |

### Sample of Rust telemetry events (behavioral, not raw prompts)
```
codex.exec_server.process_start   → program, args, env, cwd, environment
codex.exec_server.http_request    → method, server.address, server.port, status_code
codex.git.*                       → subcommand, changed_path_count, repository, branch
codex.mcp.call / codex.tool.call  → tool name, error_type, error_code, duration
codex.turn.token_usage / cost_microusd / conversation.turn.count
codex.guardian_v2.classification  → risk_level, authorization, outcome
codex.api_request, codex.apps.*, codex.hooks.*, codex.goal.*, codex.rollback.*
```
Notable: **process spawns are telemetered with `args`, `env`, `cwd`** — high
diagnostic value, but also high fingerprinting value. Tool calls, repos,
branches, URLs, and turn costs are all reported. `internal_chat_message_metadata_passthrough`
carries `content_item_kinds`, `executed_tool_calls`, `turn_id`.

## 6. Local data stores (on-disk privacy surface)
- `~/.codex/`: `sessions/`, `session_index.jsonl`, `logs_2.sqlite`,
  `memories_1.sqlite`, `goals_1.sqlite`, `queue_1.sqlite`, `history`, `computer-use`,
  `memories`, `node_repl`, `shell_snapshots`, `attachments`.
- `~/Library/Application Support/Codex/`: Chromium profile — `Cookies`,
  `Local Storage`, `Cache`, `extensions_crx_cache`, `browser-sidebar-page-states.json`,
  `codex-browser-app`, `owl-feature-bootstrap-cache.json`.
- Computer History `events.jsonl` segments (typed/selected text, URLs, titles).

## 7. Permissions the app requests
Camera, microphone, **system audio capture**, calendars, reminders,
**Apple Events (automation)**, **Input Monitoring**, Accessibility, user-selected
files. App is **not sandboxed** (`app-sandbox=false`), `NSAllowsArbitraryLoads=true`.

## 8. Privacy vectors, ranked

1. **Computer History** — continuous typed/selected text, window titles, URLs
   (opt-in, local, agent-readable). Highest sensitivity.
2. **Telemetry with process args/env/cwd + git repo/branch + URLs** — rich
   behavioral fingerprint sent to OTel/Segment/Statsig/Sentry.
3. **System-audio capture** — real capture surface; local-only claim verified in
   the driver, but entitlement is broad.
4. **Screen recording → memories** (`codex_chronicle`, closed) — screen content
   persisted as memories.
5. **Sentry breadcrumbs** — may carry file paths/commands from errors.
6. **Accessibility tree access** — full UI state of other apps during computer-use.
7. **Clipboard** — agent-invoked reads (could be prompt-injected to read).
8. **Opaque fields** — `x-openai-encrypted-tool-arguments`, `encrypted_content`
   are client-unreadable (privacy-preserving to the user, opaque anyway).

## 9. What was NOT found
- No hidden microphone recording loop.
- No general keystroke logger (modifiers only).
- No ambient clipboard polling.
- No hidden credential exfiltration (no secrets in bundle; credential fields are
  explicitly protected from automation).
- Datadog honors Do-Not-Track.

## 10. Recommendations to reduce exposure
- Turn off / tune **Computer History** (`do_not_observe` per app, or `stopped`).
- Disable **system-audio** and screen-recording features in Settings if unused.
- Set `BROWSER_USE_SECURITY_MODE` / origin policy to restrict browser automation.
- Block telemetry egress at the network layer (Sentry/Statsig/Segment/Datadog/OTel)
  if desired; `OTEL_TRACES_*`/`CODEX_OTEL_*` env vars control the OTel path.
- Revoke **Input Monitoring**/**Accessibility** if you don't use modifier triggers
  or computer-use.
