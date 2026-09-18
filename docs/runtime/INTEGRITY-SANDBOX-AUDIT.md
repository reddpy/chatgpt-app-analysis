# Integrity / attestation + sandbox enforcement — audit

## A. Integrity & attestation

### Electron fuses (read from the `Codex Framework` binary fuse wire)
| Fuse | State |
|---|---|
| `RunAsNode` | **disabled** (good — no `ELECTRON_RUN_AS_NODE`) |
| `EnableNodeOptionsEnvironmentVariable` | default (node options not honored) |
| `EnableCookieEncryption` | **enabled** |
| `EnableEmbeddedAsarIntegrityValidation` | **set** |
| `EnableNodeCliInspectArguments` | default |
| `OnlyLoadAppFromAsar` | default (not explicitly enforced) |
| `GrantFileProtocolExtraPrivileges` | default |

### ASAR integrity
`Info.plist` `ElectronAsarIntegrity` pins `Resources/app.asar` → `SHA256`
`328cf1a0d0841e8e48e86e1c1f2bf3020ab2b3c9300cadf46902fda713af5827`. Electron
verifies the asar against this at launch (defeats "edit app.asar" tampering).

### Rolling integrity-state envelope (`@oai/integrity-state`)
- `IntegrityStateStore` is backed by global-state key **`electron-integrity-state-envelope`**.
- `getRequestState()` returns the stored envelope and it is attached **only to
  OpenAI URLs** (`isDesktopAuthAllowedUrl`; *"Refusing to attach integrity state
  to non-OpenAI URL"*).
- `storeResponseUpdate(responseHeaders, …)` — the **server returns the next
  envelope in response headers**; the client stores it and echoes it on the next
  request. `{currentState, expectedState, nextState}` validates the round-trip.
- So it's a **server-issued, stateful anti-replay/tamper signal**, not a local
  secret (stored plaintext).

### Attestation (Rust + macOS hardware)
- `app-server/src/attestation.rs`: `AttestationProvider` requests an attestation
  **header value per thread/connection** (`request_attestation_header_value_with_timeout`,
  100 ms budget) — the backend can require attestation on requests.
- macOS: `devicecheck.node` (Apple **DeviceCheck**) + `remote-control-device-key.node`
  (**Secure Enclave** ECDSA P-256) + `browser-use-peer-authorization` (verifies
  the code-signing identity of Unix-socket peers via audit tokens).
- Crash breadcrumbs include `integrity-failure`.

**Assessment:** integrity is a **server-validated signal**, not a hard local
boundary. It meaningfully raises the bar (ASAR pinning, fuse hardening, rolling
envelope, device attestation) against *casual* client modification, but on a
compromised machine the app allows JIT + unsigned executable memory, so runtime
instrumentation (debugger/Frida-class) remains possible.

## B. Sandbox enforcement

### Renderer (hardened)
The main windows use
`{contextIsolation:true, nodeIntegration:false, sandbox:true, devTools:false}`
(some with `javascript:false`); **these protect the renderer from itself**. A
separate `browser-page-preload` runs for web content.

### The app itself (NOT sandboxed)
Entitlements: `app-sandbox=false`, `allow-jit`, `allow-unsigned-executable-memory`,
`device.camera`, `device.audio-input`, `audio-capture`,
`personal-information.calendars`, `automation.apple-events`, `network.client`;
ATS `NSAllowsArbitraryLoads=true`. So code execution anywhere in the app process
reaches the user's account, screen, mic, calendar, and automation.

### Tool-execution sandbox (Rust `sandboxing` crate)
- macOS **seatbelt** (dominant), Linux **bwrap** + **landlock**, Windows AppContainer.
- Modes: `read-only` · `workspace-write` · `danger-full-access`; writable roots,
  tmpdir/tmp exclusions.
- Network can be cut (`CODEX_SANDBOX_NETWORK_DISABLED`).

### Network / policy enforcement
- **Network proxy**: per-session `allowed_domains` / `denied_domains`, TLS MITM,
  **credential brokering** (sandboxed process sees dummy creds), SSRF/loopback
  and non-public-IP blocking.
- **Browser origin policy**: per-origin `access / uploads / downloads /
  fullCdpAccess / autoReview`, wildcard semantics, MDM/requirements overrides.
- **Approvals**: `AskForApproval::{OnRequest, OnFailure, UnlessTrusted, Never, Granular}`.

## C. Gaps / trust-relevant findings

1. **The app is not OS-sandboxed** while holding broad entitlements — the true
   security boundary for local actions is the **tool sandbox + approvals +
   Guardian**, all of which are logic (and Guardian is a probabilistic model).
2. **Guardian is the only barrier for many actions** (unsandboxed fallback), so
   enforcement is "model reviewing model," not a hard boundary.
3. **Runtime injection possible**: `allow-jit` + `allow-unsigned-executable-memory`
   mean ASAR pinning doesn't stop a local attacker from instrumenting the process.
4. **`OnlyLoadAppFromAsar` not explicitly enabled** (fuse default) — a local
   attacker could try to load the app from an unpacked directory.
5. **Debug/eval surfaces gated by an env var (`BUILD_FLAVOR`)** — client-side,
   trivially bypassable (demonstrated), exposing the debug panel, control window
   and eval-control harness.
6. **ATS off** (`NSAllowsArbitraryLoads`) and **`http`/`https` handler
   registration** widen the network surface.
7. Integrity envelope + global state are **plaintext on disk**; the trust relies
   on the server validating the round-trip.

## Verdict
Layered and above-average for an Electron app (fuses, ASAR pinning, rolling
integrity envelope, DeviceCheck/Secure-Enclave attestation, hardened renderer,
real tool-exec sandboxes, credential brokering, SSRF blocking). The residual risk
is the classic two: (a) the **app is unsandboxed**, and (b) enforcement of local
actions is **prompt/model-level (Guardian)**, not kernel-level — plus the
**client-side bypassable debug gate**.
