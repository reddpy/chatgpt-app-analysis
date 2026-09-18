# Reversing the Rust portion of the ChatGPT/Codex desktop app

**Bottom line:** the Rust backend did **not** need to be decompiled. The shipped
`codex` binary is **unstripped**, and its exact source is public. Recovered the
**exact source tree** for the Codex CLI (and the Code Mode host) by version
matching to upstream.

---

## Target

| Binary | Size | Source recovered? |
|---|---|---|
| `Contents/Resources/codex` | 228 MB | ✅ exact source (tag `rust-v0.155.0-alpha.2.6`) |
| `Contents/Resources/codex-code-mode-host` | 63 MB | ✅ exact source (same tag) |
| `Contents/Resources/codex_chronicle` | 4.6 MB | ❌ **internal-only**, not in public repo |
| `Contents/Resources/rg` | 4 MB | third-party (ripgrep 15.2.0) |
| Electron main/renderer JS (`app.asar`) | — | ❌ closed (minified, no source maps) |
| `native/*.node`, "owl" framework | — | ❌ closed native |

Version string baked into the binary: **`0.155.0-alpha.2.6`**
(`codex-cli`, `codex-app`, `codex-doctor`, `codex-mcp-client`, app-server-daemon).
Target: `aarch64-apple-darwin`, release, built from the `codex-rs/` workspace.

## Provenance

- Repo: `https://github.com/openai/codex` (Apache-2.0)
- Tag: `rust-v0.155.0-alpha.2.6`
- Commit: `fb8878663c35e2fbab54d797eb777a3320ffece0`
- `codex-rs/Cargo.toml` workspace `version = "0.155.0-alpha.2.6"` ✅

## Evidence the match is exact (not approximate)

1. **Crate coverage** — the binary's symbol table contains 120 distinct
   `codex_*` crates. **120/120 exist** in the public source; **0 missing.**
2. **Symbol resolution** — 7,394 unique `codex_*` function/type symbols were
   demangled and mapped to an existing source module file: **7,394/7,394 resolved.**
3. **Line-exact tracing validation** — the binary embeds `tracing` event target
   strings of the form `<crate>/<path>.rs:<line>`. Each was checked against the
   source; 3/3 sampled are line-exact:
   - `codex-mcp/src/rmcp_client.rs:558` → `warn!("failed to initialize MCP client during shutdown: {error:#}")`
   - `app-server/src/fuzzy_file_search.rs:79` → `warn!("fuzzy-file-search join failed: {err}")`
   - `thread-store/src/local/live_writer.rs:178` → `warn!("failed to project durable rollout during shutdown for {thread_id}: {err}")`

## Method (reproducible)

```sh
# 1. Is it stripped?
nm -a /Applications/ChatGPT.app/Contents/Resources/codex | wc -l   # ~24k, has T symbols
otool -l <binary> | grep -i dwarf                                   # none (no debug info)

# 2. Identify version
strings -a <binary> | rg -i '0\.155\.0'

# 3. Demangle the legacy Rust symbols
#    (symbols are legacy-mangled with $..$ escapes; llvm-cxxfilt eats length prefixes,
#     then substitute $LT$->"<", $GT$->">", $RF$->"&", $u20$->" ", ".."->"::")
nm <binary> | awk '$2 ~ /^[TtDdSsBb]$/ {print $3}' > mangled.txt
xcrun llvm-cxxfilt < <(sed 's/^_//' mangled.txt) > demangled.txt
# see demangle.mjs

# 4. Fetch the exact source
curl -L -o codex-src.tar.gz \
  https://codeload.github.com/openai/codex/tar.gz/refs/tags/rust-v0.155.0-alpha.2.6

# 5. Map symbols -> source files, compute coverage
node map-to-source.mjs
```

## Output artifacts in this folder

| File | Contents |
|---|---|
| `codex-rs/` | **exact upstream source** (4,025 `.rs` files, 85 MB) |
| `codex-src.tar.gz` | original tarball (14 MB) |
| `symbol-to-source-map.tsv` | 7,394 binary symbols → source file path |
| `codex-symbols-readable.txt` | all 22,377 demangled names (human-readable) |
| `codex-symbols-uniq.txt` | 14,490 unique symbols |
| `codex-symbols-mangled.txt` | raw symbol table extract |
| `crates-missing-from-public.txt` | empty — every `codex_*` crate is public |
| `codex-strings.txt` | extracted strings (source paths, feature flags, endpoints) |
| `demangle.mjs`, `map-to-source.mjs` | the tooling used |

## Recovered architecture

120 internal workspace crates. Largest by symbol count:
`codex_core` (1072), `codex_tui` (889), `codex_app_server` (428),
`codex_exec_server` (300), `codex_core_plugins` (220), `codex_protocol` (156),
`codex_network_proxy` (151), `codex_rmcp_client` (136), `codex_mcp` (136),
`codex_login` (126), `codex_app_server_transport` (113), `codex_config` (108),
`codex_rollout` (104), `codex_thread_store` (102), `codex_api` (98),
`codex_skills_extension` (90), `codex_analytics` (66), `codex_otel` (60),
`codex_code_mode` (43), `codex_hooks` (41), `codex_guardian_v2` (31),
`codex_cloud_config` (30), `codex_network_proxy`, `codex_connectors`,
`codex_sandboxing`, `codex_execpolicy`, `codex_apply_patch`, `codex_worktree`, …

Full list: `rg 'name = ' codex-rs/**/Cargo.toml`, or the source tree itself.
Third-party deps (365 crates in the symbol table, e.g. `tokio`, `rustls`, `h2`,
`ring`, `rama_http_core`, `rmcp`, `starlark`, `aws_sdk_*`, `gix`, `ratatui`)
are reproducible from `codex-rs/Cargo.lock` via `cargo vendor`.

Notable internal-only crates that ARE public in this tag (counter-intuitive but
confirmed): `codex_guardian_v2`, `codex_guardian_reviewer`, `codex_memories_*`,
`codex_realtime_webrtc`, `codex_workload_identity`, `codex_agent_identity`.

## What is NOT recoverable from this repo

- **`codex_chronicle`** — separate Rust binary (ScreenCaptureKit screen-recording
  → markdown "memories"), top-level crate `codex_chronicle`, **no matching crate
  in the public tag**. Closed. Only symbols/strings available (see `codex-strings.txt`).
- **Electron app** (`app.asar`) — minified JS, no source maps. See `../analysis/`.
- **Native addons / "owl" Electron fork** — compiled, no source.

## Notes / caveats

- The public repo also contains a `codex-cli/` (the npm wrapper) and `sdk/`,
  `docs/`, `bazel/` build files — all included in `codex-src.tar.gz`.
- The desktop app itself is a *different* product surface (Electron); only its
  Rust sidecar (the CLI/app-server) comes from this repo.
- Commit `fb8878…` is the exact build point; no local patches were observed
  (line-exact tracing matches). Any future app update will change the version
  string → re-run the method above with the new tag.
