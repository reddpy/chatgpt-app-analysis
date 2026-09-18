# ChatGPT / Codex desktop app — reverse-engineering dossier

Target: `/Applications/ChatGPT.app` (bundle id `com.openai.codex`, v26.911.61220,
Electron 42.3.0, internal shell codename **owl**). Rust core is public (Apache-2.0)
and was recovered as **exact source** at tag `rust-v0.155.0-alpha.2.6`; the desktop
shell, native addons, and Chrome layer were recovered by extraction/decompilation.

> **Rebuild note:** this dossier was reconstructed from the opencode session store
> after the working copy was deleted. The authored analysis below is complete; the
> large `raw/` dumps, decompiled C, and capture JSONL files are not included — see
> "Lost / reproducible".

## Structure

```
docs/
  product/         what ships + hidden/unreleased surfaces
  finance-agents/  finance, ledger, credits, Hermes agents, Rosalind
  safety/          guardrails, Guardian/Luna, prompts, moderation, privacy
  chronicle/       screen-recording → memories subsystem
  runtime/         owl framework, native/JS bridges, debug surfaces, integrity
  telemetry/       telemetry map, event taxonomy, Statsig gates, backend API
  _archive/        superseded indexes
sources/           reverse-engineering source notes / methodology
harness/           Guardian capture instrumentation
INDEX.md           this file
```

## docs/product
- `DESKTOP-FEATURE-INVENTORY.md` — full surface inventory
- `HIDDEN-FINDINGS.md`, `JUICY-FINDINGS-2.md` — headline findings
- `BAZAAR-ADS.md` — ads system ("Bazaar")
- `PET-SYSTEM.md` — pets feature
- `PLUGIN-TEMPLATE-AUTOMATIONS.md` — template automations
- `CODENAME-GLOSSARY.md` — internal codenames

## docs/finance-agents
- `AIP-LEDGER-FINANCE.md`, `PLAID-AND-LEDGER-WIDGETS.md`, `EXPERIAN-CREDIT-TRACE.md`,
  `FINANCE-SHEEP-AND-AUTOMATIONS.md`, `BILLING-CREDITS.md`
- `HERMES-AGENTS-TRACE.md`, `HERMES-AGENTS-DEEP.md`
- `ROSALIND-SCIENCE.md`

## docs/safety
- `HIDDEN-GUARDRAILS.md` — guardrail inventory
- `GUARDIAN-LUNA-AND-SECRETS.md` — Guardian ("Luna") + secret scrubbing
- `GUARDIAN-CAPTURE-HARNESS.md` — instrumented-build / proxy capture method
- `RISK-DECISION-TABLE.md` — observed reviewer verdicts
- `APP-PROMPTS-AND-MODERATION.md` — app prompts + moderation taxonomy
- `BROWSER-USE-POLICY.md`
- `PRIVACY-AUDIT.md`

## docs/chronicle
- `CHRONICLE-DEEP-DIVE.md` — `codex_chronicle`: multi-display capture, privacy filter,
  10-min/6-hour recursive summarizer, `openai-memgen` config, storage layout

## docs/runtime
- `OWL-REVERSE-ENGINEERING.md`, `OWL-VS-ELECTRON-DIFF.md`, `OWL-BUILTIN-JS-DIFF.md`,
  `OWL-JS-MAPPING.md`
- `HIDDEN-MODES-AND-ACCESS.md`, `HIDDEN-DEBUG-SURFACES.md`,
  `electronBridge-and-SentryIPC.md`, `INTEGRITY-SANDBOX-AUDIT.md`

## docs/telemetry
- `MASTER-TELEMETRY-MAP.md`, `PRODUCT-EVENT-TAXONOMY.md`, `STATSIG-ID-MAP.md`,
  `BACKEND-API-MAP.md`

## sources
- `reversing-rust.md` — Rust recovery methodology (tag → exact source)
- `asar-and-native.md` — asar/native closure notes
- `code-mode-host.md`, `feature-code.md`, `owl-framework.md`,
  `native-decompiled.md`, `native-decompiled-owl-framework.md`

## harness
- `sampler-capture.patch` — instrumentation for `guardian-v2` sampler (request + verdict)

## Lost / reproducible
Deleted with the working copy: `raw/` string/symbol dumps, `*.decompiled.c`,
`asar-extract/`, `rust-source/` tree, `prompts/`, `js-prompts/`, capture JSONL, and
the guardian decision-table capture.
Reproduce:
- `app.asar` → `/Applications/ChatGPT.app/Contents/Resources/app.asar` (re-extract)
- Rust source → re-clone `openai/codex` @ `rust-v0.155.0-alpha.2.6`
- Ghidra decompilations → re-run on the app's native binaries
- Guardian captures → rerun the recipe in `docs/safety/GUARDIAN-CAPTURE-HARNESS.md`
