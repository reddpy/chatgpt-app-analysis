# App-only hidden guardrails (browser automation service)

Source of truth: the bundled browser-automation service shipped inside the app
(NOT the open Rust core):
`Resources/plugins/openai-bundled/plugins/{browser,chrome}/scripts/browser-service.mjs`
(+ `browser-client.mjs`). This is a full CDP-based automation engine.

## 1. Credential-field protection (anti-exfiltration)

The service defines a protected-field policy and **refuses to run clipboard /
cut / paste operations against credential fields** (throws *"Virtual clipboard is
missing credential protection"* if it is absent). Exact policy:

```js
const protectedCredentialFieldPolicy = {
  attributes: ["type","autocomplete","id","name","placeholder","aria-label","title"],
  pattern: /user[-_ ]?name|e[-_ ]?mail|one[-_ ]?time[-_ ]?code|password|passcode|passwd|\botp\b|\b(?:2fa|mfa)\b|phone|mobile|\btel\b/i
};
```

It also **strips `value` attributes from hidden inputs** when copying/exporting
page content, so hidden credential values can't be harvested through the
clipboard or a page-assets bundle.

## 2. Automated safety prechecks

- `BROWSER_USE_AUTOMATED_SAFETY_PRECHECKS_ENABLED` — gates a "safety precheck"
  layer around browser actions.
- `BROWSER_USE_SECURITY_MODE` — selectable mode; one value is
  **`gaas-browser-environment`** (a locked-down environment; also
  `gaasBrowserConfig`, `gaas.browserConfig.user_id`).

## 3. Capability kill-switches (env-driven)

```
BROWSER_USE_DISABLE_AMBIENT_NETWORK
BROWSER_USE_DISABLE_API_MEMBERS
BROWSER_USE_DISABLE_BROWSER_CAPABILITIES
BROWSER_USE_DISABLE_TAB_CAPABILITIES
BROWSER_USE_FULL_CDP_ACCESS_ENABLED
BROWSER_USE_SECURITY_MODE
BROWSER_USE_AUTOMATED_SAFETY_PRECHECKS_ENABLED
```
Plus internal build/identity vars: `BROWSER_USE_CODEX_APP_VERSION`,
`BROWSER_USE_CODEX_APP_BUILD_FLAVOR`, `BROWSER_USE_BROWSER_CLIENT_BUILD`,
`BROWSER_USE_CONFIG_PATH`, `BROWSER_USE_AVAILABLE_BACKENDS`.

## 4. "TinySky" — a second internal codename

```
BROWSER_USE_TINYSKY_ENABLED
BROWSER_USE_ENABLE_TINYSKY_ACCESSIBILITY
BROWSER_USE_ACCESSIBILITY_CORE_WASM_PATH
```
"TinySky" appears to be a lightweight accessibility/DOM engine (WASM) used as an
alternative to full CDP for reading pages.

## 5. Anti-hijack input targeting

Text input is bound to a per-target nonce (`__codexIabExpectedInputTargetToken`)
verified at dispatch time ("Active element is no longer the expected input
target: target token mismatch") so automation can't be redirected to a different
focused field mid-typing.

## 6. Native clipboard blocked

Sending native clipboard shortcuts is refused —
*"Native clipboard shortcuts are disabled; use Browser Use virtual clipboard
commands instead."* — everything goes through the instrumented virtual clipboard,
which is where the credential policy is enforced.

## 7. Full command surface (from the exported command map)

`BrowserAuthHandoffCommand`, `BrowserUserClaimTabCommand`,
`BrowserUserGetTabContextCommand`, `CreateTabCommand`, `CloseTabCommand`,
`ListTabsCommand`, `CuaClick/DoubleClick/Drag/Keypress/Move/Scroll/Type*`,
`Playwright*` (locator/click/fill/press/evaluate/download/filechooser/…),
`TabAxAction`/`TabAxGetState` (accessibility tree),
`TabCdpCall`/`TabCdpEvents` (raw CDP), `TabClipboard{Read,Write,ReadText,WriteText}`,
`TabContentExport{YouTubeTranscript,GSuite}`, `TabDevLogs`, `WebMcpInvokeTool` /
`WebMcpListTools`, `TabPageAssets{Bundle,List}`, `TabScreenshot`.

Notable capability: `TabCdpCall` / `TabCdpEvents` expose raw CDP, gated by
`BROWSER_USE_FULL_CDP_ACCESS_ENABLED` + `gaas-browser-environment`.

## Why this counts as "hidden"
None of this is in the UI, docs, or the open Rust repo. It's app-only guardrail
code compiled into a bundled `.mjs` service, consisting of an explicit
credential-protection policy, safety prechecks, capability kill-switches, and
internal codenames (`gaas`, `tinysky`).
