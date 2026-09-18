# `electronBridge` + `__SENTRY_IPC__` — full API dump

Source: the app's preload (`app.asar/.vite/build/preload.js`). Exposed to the
renderer via `contextBridge.exposeInMainWorld`.

---

## `window.electronBridge`

```ts
interface ElectronBridge {
  /** constant */
  windowType: "electron";

  // ── chunked message protocol ─────────────────────────────
  /** ack a chunked message; ipc send → codex_desktop:chunked-message-ack (transferId, n) */
  acknowledgeChunkedMessage(transferId: string, n: number): void;
  /** performance.timeOrigin captured at preload start */
  getPreloadStartedAtMs(): number;

  // ── renderer → main RPC ──────────────────────────────────
  /** main RPC; ipc invoke → codex_desktop:message-from-view
   *  If message.type === "shared-object-set" it also updates the local
   *  shared-object cache: { type:"shared-object-set", key, value } */
  sendMessageFromView(message: { type: string; [k: string]: unknown }): Promise<void>;

  /** per-worker channel; ipc invoke → codex_desktop:worker:<worker>:from-view */
  sendWorkerMessageFromView(worker: string, message: unknown): Promise<void>;
  /** ipc on → codex_desktop:worker:<worker>:for-view ; returns unsubscribe */
  subscribeToWorkerMessages(worker: string, cb: (msg: unknown) => void): () => void;

  // ── files / drag ─────────────────────────────────────────
  /** Electron webUtils.getPathForFile */
  getPathForFile(file: File): string | null;
  /** ipc sendSync → codex_desktop:start-file-drag ; returns boolean */
  startFileDrag(payload: unknown): boolean;
  /** ipc send → codex_desktop:start-link-drag */
  startLinkDrag(payload: unknown): void;

  // ── menus / metrics ──────────────────────────────────────
  /** ipc invoke → codex_desktop:show-context-menu */
  showContextMenu(params: unknown): Promise<unknown>;
  /** ipc invoke → codex_desktop:get-fast-mode-rollout-metrics */
  getFastModeRolloutMetrics(params: unknown): Promise<unknown>;

  // ── shared-object snapshot ───────────────────────────────
  /** read from the local shared-object cache (seeded synchronously at preload) */
  getSharedObjectSnapshotValue(key: string): unknown;

  // ── bootstrap / theme ────────────────────────────────────
  /** ipc sendSync → codex_desktop:get-initial-sidebar-bootstrap (cached after 1st call) */
  getInitialSidebarBootstrap(): unknown;
  /** current system theme variant, "light" | "dark" */
  getSystemThemeVariant(): string;
  /** ipc on → codex_desktop:system-theme-variant-updated ; returns unsubscribe */
  subscribeToSystemThemeVariant(cb: () => void): () => void;

  // ── sentry ───────────────────────────────────────────────
  /** ipc invoke → codex_desktop:trigger-sentry-test (fires a test exception) */
  triggerSentryTestError(): Promise<void>;
  /** ipc sendSync → codex_desktop:get-sentry-init-options
   *  returns { appVersion, codexAppSessionId, ... } */
  getSentryInitOptions(): {
    appVersion: string;
    codexAppSessionId: string;
    [k: string]: unknown;
  };

  // ── misc ─────────────────────────────────────────────────
  /** "Codex Desktop/<version> (Mac OS|X11; Linux|Windows NT 10.0; <arch>)" */
  getDesktopUserAgent(): string;
  /** == getSentryInitOptions().codexAppSessionId */
  getAppSessionId(): string;
  /** ipc sendSync → codex_desktop:get-build-flavor  ("prod" | "dev" | "agent" | "nightly" | ...) */
  getBuildFlavor(): string;
  /** platform === "darwin" && arch === "arm64" */
  isDeviceCheckSupported(): boolean;
  /** platform === "darwin" && arch === "x64" */
  isIntelMacBuild(): boolean;
}
```

### Synchronous preload reads (before the page runs)
At preload time the app does `ipcRenderer.sendSync(...)` for:
- `codex_desktop:get-sentry-init-options`
- `codex_desktop:get-build-flavor`
- `codex_desktop:get-shared-object-snapshot`
- `codex_desktop:get-system-theme-variant`

### Inbound channels (main → renderer)
```ts
ipcRenderer.on("codex_desktop:message-for-view", (e, data) => {
  // if data.type === "shared-object-updated" → updates local shared-object cache
  // if data.type === "remote-hosted-pip-content-layout-state-changed" → manages VideoDecoder lifecycle
  window.dispatchEvent(new MessageEvent("message", { data }));
});

ipcRenderer.on("codex_desktop:mcp-app-sandbox-host-message", (e, msg) => {
  if (window.location.origin !== "null") window.postMessage(msg, origin, e.ports);
});

ipcRenderer.on("codex_desktop:remote-hosted-pip-video-frame", (e, frame) => {
  // decodes a video frame (VideoDecoder) and postMessage's it to the page
});

// outbound bridge for AppView RPC:
window.addEventListener("message", (ev) => {
  if (ev.source === window && ev.data?.type === "connect-app-host") {
    ipcRenderer.postMessage("codex_desktop:connect-app-host", undefined, [ev.data.port]);
  }
});

// also exposed:
contextBridge.exposeInMainWorld("codexWindowType", "electron");
```

### IPC channel reference (preload)
```
codex_desktop:chunked-message-ack
codex_desktop:remote-hosted-pip-video-frame
codex_desktop:mcp-app-sandbox-host-message
codex_desktop:show-context-menu
codex_desktop:get-sentry-init-options
codex_desktop:get-build-flavor
codex_desktop:get-system-theme-variant
codex_desktop:get-shared-object-snapshot
codex_desktop:get-initial-sidebar-bootstrap
codex_desktop:get-fast-mode-rollout-metrics
codex_desktop:system-theme-variant-updated
codex_desktop:trigger-sentry-test
codex_desktop:connect-app-host
codex_desktop:start-file-drag
codex_desktop:start-link-drag
codex_desktop:message-from-view
codex_desktop:message-for-view
codex_desktop:worker:<worker>:from-view
codex_desktop:worker:<worker>:for-view
```

---

## `window.__SENTRY_IPC__["sentry-ipc"]`

Installed by the bundled Sentry Electron preload. All are fire-and-forget
`ipcRenderer.send` on the Sentry IPC namespace.

```ts
interface SentryIpc {
  /** ipc send → sentry-ipc.start */
  sendRendererStart(): void;
  /** ipc send → sentry-ipc.scope           (Sentry scope payload) */
  sendScope(scope: unknown): void;
  /** ipc send → sentry-ipc.envelope        (Sentry envelope payload) */
  sendEnvelope(envelope: unknown): void;
  /** ipc send → sentry-ipc.status          (Sentry client status) */
  sendStatus(status: unknown): void;
  /** ipc send → sentry-ipc.structured-log */
  sendStructuredLog(log: unknown): void;
  /** ipc send → sentry-ipc.metric */
  sendMetric(metric: unknown): void;
}

window.__SENTRY_IPC__              // { "sentry-ipc": SentryIpc, ... }
```

Namespacing helpers used internally: `createKey(k) => "sentry-ipc."+k`,
`createUrl(host) => "sentry-ipc://"+host+"/sentry_key"`, `urlMatches(url,host)`.

---

## Quick console probes
```js
Object.keys(electronBridge)
electronBridge.getBuildFlavor()            // "dev"
electronBridge.getSentryInitOptions()      // Sentry DSN/release/session
electronBridge.getInitialSidebarBootstrap()
Object.keys(window.__SENTRY_IPC__)         // ["sentry-ipc"]
window.__SENTRY_IPC__["sentry-ipc"].sendRendererStart()
electronBridge.sendMessageFromView({ type: "<one of the 524 RPC verbs>" })
```
