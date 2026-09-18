# Feature code — extracted & de-minified

These are the actual, prettified source chunks for hidden/unreleased desktop
features, pulled straight from the shipped renderer bundle
(`webview/assets/<feature>-<hash>.js`). No decompilation needed — it's the app's
own JavaScript, just minified.

Files are `<chunk>.pretty.js` (output of `prettier --parser babel`).

## What each feature is

### appgen — generate & publish apps/sites
| File | Role |
|---|---|
| `appgen-page.pretty.js` | main page; lists the user's generated apps/sites. Imports `start-appgen-conversation`, `appgen-access`, `open-appgen-share-dialog`, `sites-default-thumbnail`, `appgen-access-state-messages` |
| `start-appgen-conversation.pretty.js` | **starts an agent conversation to generate the app** — confirms "appgen" is the coding agent + a hosting UI, not a separate builder |
| `appgen-database-page.pretty.js` | manage a generated app's database |
| `appgen-settings-page.pretty.js` | app/site settings (largest chunk, 92 KB) |
| `appgen-share-dialog.pretty.js` | sharing/publishing dialog |
| `appgen-access.pretty.js`, `appgen-access-state-messages.pretty.js` | entitlement/plan gating + messaging |
| `sites-default-thumbnail.pretty.js` | site thumbnails / site state |

### design editor — live-page editing/annotation
`design-editor-state.pretty.js` — React state + CSS-module styling for the
in-browser **design overlay** (numeric/color inputs, design modifier key,
"scrub" adjustments). Drives `browser-sidebar-design-overlay-*`.

### avatar overlay — companion/mascot
| File | Role |
|---|---|
| `avatar-overlay-quick-chat-bar.pretty.js` | the quick-chat bar UI in the overlay (106 KB) |
| `avatar-overlay-native-page.pretty.js` | the overlay's own page (395 KB — the biggest feature chunk) |
| `use-avatar-options.pretty.js` | fetches `avatarDirectory` + avatar options |

### appshots — screen capture
`appshots-settings.pretty.js` — settings for the appshots capture feature.

### Codex Micro (hardware)
| File | Role |
|---|---|
| `codex-micro-bridge.pretty.js` | bridge between the device and the app (webview/electron commands) |
| `codex-micro-commands.pretty.js` | maps device inputs → app commands (`newThread`→`newTask`, `recentThread1..6`) |
| `codex-micro-mini-games.pretty.js` | **the device ships mini-games**: `brick-breaker`, `asteroids`, `snake` |

The mini-games chunk contains real game logic (joystick angles, activation
thresholds, per-thread slot status) — so the Codex Micro controller has a play
mode, not just agent controls.

## Notes
- The chunks are thin wrappers; deep shared logic lives in
  `app-initial-<hash>.js` (9.7 MB), whose prettified form is in
  `../analysis/pretty/` (local) — symbols there are minified.
- UI copy is inline in some chunks but i18n-keyed in others (there is **no**
  English locale file; English is the source language).
- API calls/endpoints for these features sit in shared query modules in the big
  bundles, not in the feature chunks.
