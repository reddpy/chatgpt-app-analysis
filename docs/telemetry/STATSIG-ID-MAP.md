# Statsig ID → feature map

Statsig gates/experiments/configs are referenced by **numeric IDs** in the app,
and **the ID→name mapping lives server-side** (not in the bundle), so this map is
**inferred from each ID's usage context** (assigned variable + nearest
feature-like string). Raw table: `STATSIG-ID-MAP.tsv` (362 IDs).

Method: `checkGate("<id>")` returns a boolean; `dynamicConfig("<id>").get("key")`
returns a config value. But many 6–10 digit literals in the bundle are noise
(sizes, timestamps, enums), so treat hints as **likely**, not certain.

## Ads / promotions (the "bazaar" gate)
| ID | Kind | Meaning |
|---|---|---|
| `665873859` | gate | **ads-controls visibility** (`checkGate("665873859")`); must be true |
| `1648954249` | dynamic config | keys seen: `suppress_ads`, `suppress_upsell`, `promo_campaign` — the suppression config for ads/upsells |

Ads show only if `gate 665873859 === true && config 1648954249.suppress_ads !== true`.

## Mapped gates/configs (notable)
| ID | Feature hint |
|---|---|
| 2128165686 | `browse_enabled` |
| 3000193894 / 637432221 | `appgen` |
| 1498636858 / 2171042036 / 2327881676 / 2460218704 / 2791276931 / 2957382457 / 1274567217 | `chatgpt.quick-chat` |
| 1506311413 / 2212532336 / 337408568 / 741821319 / 510816968 | `computer_use` / `computer-use-frontmost-window` |
| 188145323 / 1892382740 / 2484414311 / 2633172918 / 2958834185 / 2982604767 / 3079718369 / 4131705479 / 4263582812 / 1711474471 / 2493948789 / 3947336106 | `browser.in-app` |
| 2096615506 / 2975983127 / 581682073 | `codex-large-internal-plugins` |
| 2373428540 / 3203613120 | `claude-code` |
| 1115442235 / 2256010998 | `remote_wsl_connections` |
| 1042620455 | `set-remote-control-connections-enabled` |
| 1174206456 | `suppress_upsell` |
| 1193530394 | `realtime-voice-config-override` |
| 1326334369 / 2893692197 | `show_scheduled_tab` |
| 1535340810 | `paste_threshold` |
| 1903862752 | `paste-file-support` |
| 1925940714 | `optimize_prepare_on_stream_end_v1` |
| 224248639 | `web_file_library_enabled` |
| 2493948789 | `in_app_updates` |
| 2153867414 | `sidebar_project_hover_card_install_remote_codex` |
| 2347841422 | `local_business` |
| 2910064124 | `app_block` |
| 2929091547 | `demo-tools` |
| 3194776735 | `client_defined_widget` |
| 3839945238 | `google-drive` |
| 410262010 | `browser_use` |
| 419294242 | `config-requirement-disabled` |
| 576820092 | `dictation.global` |
| 72216192 | `enable_i18n` |
| 1372061905 | `hotkey-window-hotkey-state` |
| 1848317837 | `avatar-overlay-open-state-request` |
| 2126931955 / 3398492218 | `connector_openai_wallet` |
| 107580212 | `wsl-bash-availability` |
| 4027330399 | `enable_free_go_usage_settings` |
| 1062292446 | `exposure_threshold_percent` (experiment) |
| 3207467860 | `not-detected` |

## How to use
```js
const s = __STATSIG__.firstInstance;
s.checkGate("665873859")               // ads-controls gate
s.overrideGate("665873859", true)      // force ads controls visible
s.getConfig("1648954249")?.value       // suppression config (suppress_ads/suppress_upsell/promo_campaign)
```
Some gates also appear as `dynamicConfig("<id>").get("<key>")`; the key names
(the third column) are often the real feature name.

## Caveats
- Names are **inferred**; Statsig resolves names server-side, so there's no
  authoritative client map.
- The TSV includes many false-positive numeric literals — cross-check the hint
  column before trusting an entry.
