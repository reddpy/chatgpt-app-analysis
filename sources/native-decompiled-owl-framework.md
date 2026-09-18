# owl framework — targeted native decompilation

The full `Codex Framework` binary is 258 MB with **209 MB of `__text`** (~55M
instructions). Full Ghidra analysis/decompilation is not feasible on a 16 GB
machine (it would OOM and produce many GB of mostly-Chromium C). So this is a
**targeted** extraction of the OpenAI-authored owl code.

## Method

1. Extracted all owl-specific string addresses from the binary
   (`../owl-framework/owl-string-addresses.txt`, 99 targets).
2. Decoded every ARM64 `ADRP`+`ADD` pair in `__text` with a numpy bit-pattern
   scanner to find code that references those strings → **315 xrefs**.
3. Clustered the xrefs → **50 address ranges** (~1.5 MB total).
4. Scanned `__text` for `RET` boundaries to recover candidate function starts,
   then selected the **209 functions** containing owl xrefs.
5. Ghidra (import without auto-analysis) disassembled + decompiled exactly those
   209 functions → `Codex Framework.owl.c`.

## Result

| Metric | Value |
|---|---|
| owl-specific strings targeted | 99 |
| code xrefs found | 315 |
| address ranges | 50 (~1.5 MB) |
| functions decompiled | **209 / 209** |
| pseudo-C produced | 1.7 MB |

## Files

- `Codex Framework.owl.c` — pseudo-C for the 209 owl functions
- `Codex Framework.owl.funcs.txt` — index (`addr  name  size`)
- `owl-xrefs.txt` — code address → owl string (which file/feature each belongs to)
- `owl-func-starts.txt`, `owl-ranges.txt` — addresses/ranges analyzed
- `DumpFuncs.java` — the Ghidra script used

## Caveats

- Auto-analysis was **skipped** (too expensive), so functions are `FUN_<addr>`
  and cross-calls show as `func_<addr>`. The decompilation is structurally
  correct but types/names are unresolved.
- This captures functions that reference owl `CHECK`/log strings (the 35 added
  files' code paths). owl functions without such strings are not included.
- Rust-style exact source is impossible here — this is the Electron/Chromium C++
  layer, which has no public source for the owl delta.

## Key addresses

| Feature | first xref |
|---|---|
| `owl_websocket_interceptor` | 0x3f74e54 |
| `child_frame_screenshot_capture.cc` | 0x402f608 |
| `reparented_surface_capture.cc` | 0x4030fd0 |
| `inner_web_contents_capture.cc` | 0x4032df0 |
| `stuck_web_request_reporter.cc` | 0x404f6cc |
| `browser_profile_importer.cc` | 0x40973bc |
| `electron_download_history.cc` | 0x40b2428 |
| `electron_web_request_websocket.cc` | (see owl-xrefs.txt) |
| `native_glass_controller.cc` | (see owl-xrefs.txt) |
| `owl_notification_presenter_bridge_state.cc` | (see owl-xrefs.txt) |
