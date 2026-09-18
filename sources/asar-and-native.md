# Attaining the asar / Electron / native side — what's actually possible

Short answer: **no exact source** for the Electron side (unlike Rust). There are
no source maps anywhere and the app shell is closed-source, so the ceiling is
*semantic reconstruction*, not original TypeScript. Native binaries yield their
**interface** (classes/methods/properties) but not their **code**.

---

## 1. Electron main process (`app.asar/.vite/build/*.js`)

| Property | Result |
|---|---|
| Bundle format | Rolldown (Vite 8), minified, single-file chunks |
| Source maps | **None** — 0 `.map` files, no inline `data:` maps, no `sourcesContent` |
| Original file paths | None (only a synthetic `/Users/alice/project/src/main.ts`) |
| Unbundling | **webcrack cannot split it** (Rolldown output, not webpack) |

**Attainable**
- Full string/constant recovery: IPC channels, deep links, OAuth flow, endpoints,
  env vars, feature flags, error messages, user-facing logic.
- Readable control flow after `prettier --parser babel` (see `../analysis/pretty/`).
- Dependency graph and module grouping (semantic chunk names, e.g. `app-initial`, `rpc`).
- Behavior-level reimplementation is feasible from the de-minified code.

**Not attainable**
- Original TypeScript, original identifiers, module boundaries, type annotations.
- Anything only present in compile-time types (erased).

Fidelity: **~semantic / behavior**, not source.

## 2. Renderer (`app.asar/webview/`)

Same story: 10,736 minified chunks, React 19, no maps. Attainable = routes, API
paths, Statsig gate IDs, third-party services, component strings, wasm purpose.
Not attainable = original TSX/components. Fidelity: **semantic**.

## 3. Native addons & Swift helpers (`Resources/native/*`)

Symbol tables are **partially stripped** (no Rust-style recovery), but Objective-C
and Swift *runtime metadata* survives:

| Binary | ObjC methods recovered | Notes |
|---|---|---|
| `sky.node` | **460** | full `-/+ [Class selector:]` signatures + 213 Swift property names |
| `sparkle.node` | 49 | wraps `SPUUpdater` / `SPUStandardUserDriver` |
| others | 0 | C++/N-API, no ObjC surface (symbol-stripped) |

Recoverable classes include `PIPStack{Window,Host,Item,Anchor,Controller,ContentView,
DragInteraction,ResizeInteraction,ProgrammaticMove,ItemMotion,ItemCompletionEffect}`,
`RemoteHostedPIPContent{Service,Presentation,Attachment}`, `SkyFileDragSource`.

**Attainable:** Objective-C class list, method signatures/selectors, Swift stored
property names, Swift type metadata, imported framework APIs (AppKit, CoreAnimation,
CoreGraphics, ScreenCaptureKit, Security/SecKey Secure Enclave, IOKit HID, XPC).

**Not attainable from metadata:** method implementations. Those require a
disassembler/decompiler (Ghidra/IDA/Binary Ninja) and will yield pseudo-C at best,
not Swift/ObjC source.

Fidelity: **interface high, code none.**

## 4. The "owl" framework (`Codex Framework.framework`)

It is **stock Electron 42.3.0** (Chromium 153.0.8010.36) renamed (identifier
`com.openai.codex.framework`). Therefore:
- Its "source" is **public** (electron/electron + chromium).
- The only proprietary part is OpenAI's **patch delta**, recoverable by diffing the
  framework against the official Electron 42.3.0 build (symbols/strings/Info.plist).
  Known owl markers: `owl_websocket_interceptor`, `owl_electron_permission_profile_data`,
  `owl-scoped-user-agent-prefix`, `owl-chrome-scheme`, `owl-feature-bootstrap-cache.json`.

Fidelity: **upstream source + small recoverable delta.**

## 5. `codex_chronicle` (internal Rust)

Not in the public `openai/codex` repo. Symbol table present (crate `codex_chronicle`,
~199 symbols) + strings, but **no source**. Decompilation (Ghidra) → pseudo-Rust/C.

## 6. Community "rebuild"

`github.com/Haleclipse/CodexDesktop-Rebuild` is **not source** — its project
structure is literally `src/.vite/build/` + `src/webview/`, i.e. it repackages the
shipped bundles into a cross-platform Electron wrapper. Confirms the wall.

---

## Why the asymmetry?

- OpenAI open-sourced the **Rust** components (CLI/app-server) → exact source obtainable.
- The **Electron desktop shell** is closed-source, built from a private monorepo
  (`packagedFrom: /Users/runner/work/openai/openai/codex/codex-apps/electron/...`)
  and shipped with source maps stripped.

## Files in this folder

- `native-recovery/*.objc-methods.txt` — Objective-C method signatures per binary
- `native-recovery/symbols.txt` — consolidated ObjC/Swift/property name dump
- `decomp/preload/` — webcrack deobfuscation of the preload (single file; demonstrates
  that unbundling is not possible)
- `../analysis/pretty/` — full prettified main/worker/preload bundles (readable JS)

## If you want to push further

1. **Ghidra** the native `.node` addons + `codex_chronicle` → pseudo-code (hours–days, low fidelity).
2. **Diff owl vs official Electron 42.3.0** → isolate OpenAI's patch set precisely.
3. **Semantic reimplementation** of the Electron shell from the de-minified JS
   (the only route to "source" for that layer, and it's a rewrite, not a recovery).
