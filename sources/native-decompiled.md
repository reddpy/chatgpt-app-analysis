# Native addons — Ghidra decompilation

All native components of the ChatGPT/Codex macOS app were analyzed in **Ghidra
12.1.3** (headless) and decompiled to pseudo-C.

## Corpus

| Binary | Functions | Decompiled C (lines) | What it is |
|---|---|---|---|
| `sky.node` | 3,896 | 99,870 | "Sky" macOS integration + Picture-in-Picture stack (Swift/ObjC++), 460 ObjC methods |
| `codex_chronicle` | 8,415 | 582,614 | internal Rust screen-recording→memories pipeline |
| `sparkle.node` | 571 | 12,525 | Sparkle updater wrapper (ObjC++) |
| `bare-modifier-monitor` | 505 | 11,978 | global NSEvent bare-modifier monitor (Swift) |
| `launch-services-helper` | 358 | 11,367 | LaunchServices/Dock helper (Swift) |
| `remote-control-device-key.node` | 252 | 6,448 | Secure Enclave ECDSA P-256 device key (C++) |
| `system-audio-spectrum` | 232 | 5,504 | CoreAudio spectrum (Swift) |
| `hid-topology-watcher.node` | 215 | 5,895 | IOHID topology / Codex Micro (C++) |
| `devicecheck.node` | 166 | 3,991 | Apple DeviceCheck attestation (C++) |
| `browser-use-peer-authorization.node` | 159 | 3,825 | code-signature peer verification (C++) |
| `input-monitoring-permission.node` | 83 | 1,731 | TCC input-monitoring check (C++) |
| **TOTAL** | **14,852** | **745,748** | |

Only **1 function** failed to decompile (in `codex_chronicle`).

## Files per binary (`<name>.*`)

- `.functions.txt` — `address \t name \t signature \t size`
- `.functions.demangled.txt` — same, with Itanium C++ symbols demangled
- `.decompiled.c` — full pseudo-C for every function
- `.strings.txt` — extracted string literals with addresses

## How it was produced

1. The shipped Ghidra release ships **no macOS decompiler binary** (only
   `linux_x86_64/` and `win_x86_64/`). The arm64 `decompile` + `sleigh` were
   **built from the matching source** (`Ghidra_12.1.3_build` tag):
   ```sh
   make -f Makefile ghidra_opt sleigh_opt OS=Darwin MAKE_STATIC= \
        ARCH_TYPE="-arch arm64" ADDITIONAL_FLAGS="-w" OSDIR=mac_arm_64 -j8
   cp ghidra_opt  <ghidra>/Ghidra/Features/Decompiler/os/mac_arm_64/decompile
   cp sleigh_opt  <ghidra>/Ghidra/Features/Decompiler/os/mac_arm_64/sleigh
   ```
2. Headless import + analysis + a custom post-script (`DecompileAll.java`) that
   walks every function, decompiles it, and dumps functions, C, and strings:
   ```sh
   analyzeHeadless <proj> native -import <dir> -scriptPath <scripts> \
       -postScript DecompileAll.java <outdir> -analysisTimeoutPerFile 3600
   ```

## Reading the output

- **Names are mostly preserved.** ObjC methods come through verbatim
  (e.g. `-[PIPStackAnchor initWithPoint:alignment:]` → `initWithPoint:alignment:`);
  Swift helpers keep meaningful selectors. C++ symbols are Itanium-mangled —
  use the `.demangled.txt` index (`Napi::Function::New`, etc.).
- **Rust** (`codex_chronicle`) decompiles into large inlined functions; expect
  `FUN_xxxx` for local closures and heavy inlining. 8,415 functions recovered
  despite stripping.
- Ghidra's C is *pseudo-code*: types are `undefined8`/`long`, variable names are
  synthetic, and control flow is approximate. It is useful for understanding
  behavior, not for recompiling.

## Caveats

- These are **machine-generated** decompilations, not original source.
- Swift decompilation quality is lower than C/C++ (Ghidra is not Swift-aware).
- `codex_chronicle` is a Rust binary with no public source; the decompilation is
  the only code-level view available.
- The 228 MB `codex` CLI was intentionally **not** decompiled — its exact source
  is already recovered (see `../rust-source/`).
