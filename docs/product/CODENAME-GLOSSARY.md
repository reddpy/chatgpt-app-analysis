# Codename glossary

Codenames found across the binary, the owl framework, the app asar, and the Rust
core. Useful for reading the rest of the artifacts.

| Codename | Where | Meaning |
|---|---|---|
| **owl** | framework, paths, `owl-*` JS | the forked runtime — Electron shell renamed `electron/shell`→`owl/`, plus a full Chrome browser layer |
| **aperitif** | `libaperitif.dylib`, `Codex (Aperitif*).app` | Chromium **seatbelt/macOS sandbox** components renamed |
| **sky** | `sky.node`, `SkyFileDragSource` | macOS integration layer + Picture-in-Picture stack |
| **tinysky** | `BROWSER_USE_TINYSKY_*` | lightweight accessibility/DOM engine (WASM) used instead of full CDP |
| **walnut** | `Walnut.wasm`, "Walnut Exporter" | document engine (docx/xlsx/pptx authoring) |
| **wham** | `wham/app/appcast`, `/backend-api/wham/*` | backend service namespace (appcast, tasks, usage, remote control) |
| **estuary** | `/backend-api/estuary/content` | content/attachment service |
| **skysight** | `BROWSER_USE…skysight`, memories path | computer-history / screen→memory pipeline (Chronicle) |
| **chronicle** | `codex_chronicle` binary | the screen-recording→memories agent (only closed Rust binary) |
| **maitai** | Debug panel section | reactive state inspector (signals/queries/mutations) |
| **gaas** | `gaas-browser-environment`, `gaasBrowserConfig` | locked-down browser-automation security mode |
| **cua** | `cua_node`, `cua_repl` | Computer-Use Agent runtime |
| **guardian** | Rust crates + prompts | the reviewer model / safety engine |
| **beacons** | `CODEX_BEACONS_*` | in-app survey/feedback prompts |
| **bem** | `bem-*` events | presentation/scoring event system |
| **rosalind** | `CODEX_ROSALIND_*`, `chatgpt-rosalind-hero` | unreleased enrollment/announcement product |
| **atlas** | `atlas_mode_enabled`, `['atlas','chrome']`, `atlasv1` tokens | browser family + token format + mode gate |
| **tatertot** | `is_tatertot_enabled`, artwork sets | illustration/artwork theme set (study) |
| **m3m** | `m3m_workspace_enabled`, `m3mEnabled/OptedOut` | workspace-level feature/model |
| **bazaar** | `bazaar_personalization_*`, `free_ads_opt_out` | store/marketplace with personalization + ads |
| **sheep** | `FinanceSheepApiClient`, `useSheep()`, `x-openai-use-sheep: 1` | **the personal-finance backend gateway** (flag `chatgpt-ledger-widgets-sheep-fastapi`); header sent on finance calls |
| **aip / ledger** | `/aip/ledger/*`, route `/finances` | personal-finance product (Plaid bank links, credit, financial memories, widgets) |
| **hermes** | `/hermes/*`, "Workspace Agents" | agent builder + RRULE scheduler platform |
| **amphora** | `/amphora/*` | workspaces (members, invites, permissions) |
| **hazelnuts** | `/hazelnuts/*` | generic backend objects |
| **celsius / snorlax / olympic** | `/celsius/ws/user`, `/gizmos/snorlax/sidebar`, `/aip/olympic/*` | misc service codenames |
| **quicksilver / avas** | `/wham/realtime/calls?intent=quicksilver&architecture=avas` | realtime-voice call architecture |
| **cme / picture_v2 / search** | artwork sets | theme/illustration groupings (siblings of tatertot) |
| **luna / terra / sol** | model slugs `gpt-5.6-*` | unreleased model codenames |
| **codex micro** | `codex-micro-*`, `hid-topology-watcher` | the hardware controller (Work Louder) + its mini-games |
| **owl feature flags** | `owl-feature-bootstrap-cache.json` | OwlHistory, OwlPrinting, OwlExtensions, OwlOpenAIGoLinks |
