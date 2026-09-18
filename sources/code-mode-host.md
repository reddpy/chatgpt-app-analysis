# `codex-code-mode-host` — exact source recovered

Same story as the `codex` CLI: the binary is **unstripped** and its version
string is **`0.155.0-alpha.2.6`**, which is a public tag in `openai/codex`.

Recovered the exact source for every internal crate it contains. The binary also
embeds **V8** (~18k symbols) and **Starlark** for Code Mode execution — those are
third-party and reproducible from `Cargo.lock`.

## Crate coverage

| Crate | Symbols | Public source (tag `rust-v0.155.0-alpha.2.6`) |
|---|---|---|
| `codex_code_mode_host` | 38 | `codex-rs/code-mode-host/src/{main,lib,delegate,peer,transport}.rs`, `grpc/` |
| `codex_code_mode_runtime` | 23 | `codex-rs/code-mode-runtime/` |
| `codex_code_mode_protocol` | 7 | `codex-rs/code-mode-protocol/` |
| `codex_execpolicy` | 7 | `codex-rs/execpolicy/` |
| `codex_otel` | 6 | `codex-rs/otel/` |
| `codex_otel_trace_websocket` | 6 | `codex-rs/otel/` |
| `codex_utils_absolute_path` | 4 | `codex-rs/utils/absolute-path/` |
| `codex_utils_string` | 1 | `codex-rs/utils/string/` |

**8/8 crates matched, 125/125 codex symbols resolved to a source file.**

## Files

- `cmh-symbol-to-source-map.tsv` — binary symbol → source file
- `cmh-symbols-readable.txt` / `cmh-symbols-uniq.txt` — demangled symbol lists

## Get the source

```sh
curl -L -o codex-src.tar.gz \
  https://codeload.github.com/openai/codex/tar.gz/refs/tags/rust-v0.155.0-alpha.2.6
tar xzf codex-src.tar.gz
ls codex-src-*/codex-rs/code-mode-host/src/
```

(The full source tree is already extracted at `../rust-source/codex-rs/`.)
