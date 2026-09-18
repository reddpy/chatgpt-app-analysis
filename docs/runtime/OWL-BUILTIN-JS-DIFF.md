# Built-in JS diff: owl vs stock Electron 42.3.0

Extracted the embedded Electron built-in JS bundles (plaintext webpack) from both
frameworks and diffed module-by-module.

- stock: 8 bundles, owl: 7 bundles — 150 shared modules each.
- 69 modules **identical**; 73 "changed" — but most changed modules are pure
  **minifier noise** (verified: e.g. `./node_modules/ieee754/index.js` is the same
  code with different variable names).
- Ranking by **size delta** isolates the real owl edits.

## owl's real built-in-JS additions (by size delta)

| Δ bytes | module | what owl added |
|---|---|---|
| +3726 | `./lib/browser/guest-view-manager.ts` | `claimAdoptedWebContents`, `claimWebViewAdoptionLease`, `transferSecurityPreferences`, `getWebViewRole`, `webviewRole`, `electron_browser_web_contents` binding, lease validation |
| +2959 | `./lib/renderer/web-view/web-view-impl.ts` | `adoptWebContents`, `owl-webcontents-adoption-lease`, `owl-webcontents-adopted-web-contents-id`, `-webview-transferred` / `web-contents-transferred` events, `webviewrole=tab` |
| +1245 | `./lib/renderer/web-view/guest-view-internal.ts` | IPC channels `GUEST_VIEW_MANAGER_TRANSFER_GUEST`, `GUEST_VIEW_MANAGER_ATTACH_GUEST`, `GUEST_VIEW_MANAGER_DESTROY_GUEST`; `FinalizationRegistry` cleanup |
| +573 | `./lib/browser/default-menu.ts` | owl-customized default menu (Help submenu: Documentation / Community Discussions / Search Issues) |
| +459 | `./lib/renderer/web-view/web-view-element.ts` | role-aware webview element |
| +376 | `./lib/browser/api/web-contents.ts` | supporting changes |
| +322 | `./lib/common/init.ts` | minor |
| +261 | `./lib/browser/guest-window-manager.ts` | supporting changes |

**Theme: owl's only substantive Electron patch is a WebContents
adoption/transfer mechanism** — moving a live `WebContents` between `<webview>`
elements with a security "lease", new IPC channels, a `webviewrole="tab"`
concept, and `web-contents-transferred` events. This is what powers the app's own
tab UI (adopting browser tabs).

## owl-only built-in JS module: `native_glass_renderer`

A brand-new js2c bundle not present in stock Electron (`electron/js2c/native_glass_renderer`).
Full source recovered (1133 bytes):

```js
(function installNativeGlassRenderer(root) {
  function installNativeGlass(nativeGlass, windowObject) {
    const documentObject = windowObject?.document;
    if (!documentObject || windowObject.top !== windowObject) return;
    nativeGlass.onSupportChange((enabled) => {
      const applySupport = () => {
        const wasActive =
          documentObject.documentElement?.getAttribute("data-owl-native-glass") === "active";
        documentObject.documentElement?.setAttribute(
          "data-owl-native-glass", enabled ? "active" : "fallback");
        if (!enabled) {
          if (wasActive) windowObject.requestAnimationFrame(() => nativeGlass.fallbackReady());
          else nativeGlass.fallbackReady();
        }
      };
      if (documentObject.documentElement) applySupport();
      else documentObject.addEventListener("DOMContentLoaded", applySupport, { once: true });
    });
  }
  if (typeof nativeGlass === "object" && nativeGlass) installNativeGlass(nativeGlass, root);
})(typeof globalThis === "undefined" ? this : globalThis);
```

Renderer-side support detector for macOS **native glass** (`data-owl-native-glass`
attribute + `nativeGlass.onSupportChange` / `fallbackReady`), paired with the
native `OwlNativeGlass` classes and `owlNativeGlass*` bindings found earlier.

## Conclusion
owl's Electron-layer JS modifications are **small and focused**:
1. WebContents adopt/transfer + adoption lease (+~8 KB across 5 webview modules).
2. A new `native_glass_renderer` bundle (1.1 KB) for Apple glass support.
3. Cosmetic default-menu + `common/init` tweaks.

Everything else in the built-in JS matches stock Electron 42.3.0. So the
"Electron layer" of owl is nearly stock; the real owl-specific surface is the
**native `owl/` lib (93 paths, 35 added files)** + Chrome browser layer + the
app's own asar.
