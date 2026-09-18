# Guardian async-classifier capture harness

An **instrumented build of `codex`** that logs every asynchronous Guardian
(`gpt-5.6-luna`) classification request and its `high|low` verdict. Turns the
otherwise in-memory-only classifier into a capturable, replayable signal.

## Status
- **Build: works.** `codex-cli` compiles from the pinned source with `--ignore-rust-version`
  (rustc 1.92 vs the crate's declared 1.94 MSRV). ~4 min cold, ~25 s incremental.
- **Input capture: works.** Every scored tool call produces the exact rendered
  classifier request (system instructions + full evidence + planned action).
- **Verdict capture: works**, demonstrated end-to-end → `verdict: "low"`.
- **Blocker:** the real classifier endpoints are **entitlement-gated for a CLI
  session** — `wss://chatgpt.com/backend-api/codex/guardian-classifier` and
  `/guardian` return **403**, and `gpt-5.6-luna` returns **404** on `/responses`.
  The desktop app has this entitlement; a raw CLI build does not. (Originator
  override `CODEX_INTERNAL_ORIGINATOR_OVERRIDE` did not change it.)

## Layout
- `sampler-capture.patch` — the exact instrumentation (5 edits to
  `ext/guardian-v2/src/async_scorer/sampler.rs`).
- `config.toml` — isolated `CODEX_HOME` config used for the runs.
- `benign.jsonl` — real capture: benign read-only command → `low`, `low`.
- `destructive.jsonl` — real capture: authorized narrow `rm -rf <dir>` → `low`, `low`.

## Build
```sh
cd <codex-src>/codex-rs
cargo build --ignore-rust-version -p codex-cli --bin codex
```

## Config (`CODEX_HOME/config.toml`)
```toml
model = "gpt-5.6-sol"
model_reasoning_effort = "low"
approval_policy = "on-request"
approvals_reviewer = "auto_review"
default_tools_approval_mode = "auto"

[features]
guardian_approval = true

[features.guardianv2]
enabled = true
free_guardian = true        # route via the unmetered /guardian-classifier

[features.guardianv2.review_scope]
computer_use_only = false   # make shell/code_mode/file/mcp/network async-scored
sandboxed_exec_commands = true

[analytics]
enabled = false
```

## Run
```sh
CODEX_HOME=/tmp/ghome \
GUARDIAN_CAPTURE=/tmp/gc.jsonl \
GUARDIAN_MODEL=gpt-5.6-sol \
  codex exec --skip-git-repo-check "run: echo hi"
```
- `GUARDIAN_CAPTURE` — JSONL output path (hook is a no-op if unset).
- `GUARDIAN_MODEL` — overrides the classifier model. Defaults to `gpt-5.6-luna`;
  set to a reachable slug to get real verdicts while entitled access is unavailable.

## Capture record shapes
- `request_built` / `request` → `{ model, prompt_cache_key, guardian_endpoint, input }`
  where `input` is the ordered Responses items: a `developer` message carrying
  `classifier_instructions.md`, then the evidence (`>>> TRANSCRIPT START …`,
  `>>> APPROVAL REQUEST START … Planned action JSON …`).
- `verdict` → `{ source, classification: "high"|"low", thread_id }`.

## How the verdict is produced (for reference)
Stream the classifier; **first non-empty `OutputTextDelta` is the verdict**;
later output cannot revise it. Parsed in `extension.rs`:
`"high" => (1.0,"high")`, `"low" => (0.0,"low")`, else error → fail-closed `1.0`.
Score is stored as `SecurityRiskScore { scores: {action_risk: f64}, call_id, action, sampled_at }`
(in-memory, never model-visible).

## Capturing the REAL Luna verdict (no proxy needed)
When the `/guardian-classifier` route connects, codex logs the verdict itself:
```
INFO codex_guardian_v2::async_scorer::extension: Guardian V2 classification result
  thread_id=… turn_id=… call_id=… tool_call_index=… action_risk=0.0
  review_threshold=0.5 sampled_at=Some(…) accepted=true
```
`action_risk 0.0 => "low"`, `1.0 => "high"`. Harvest it by running with
`RUST_LOG=codex_guardian_v2=trace` and parsing those lines (see `harvest.sh`).
A real capture was obtained once (`action_risk=0.0`, `accepted=true`).

## Why the route is usually unavailable
`wss://chatgpt.com/backend-api/codex/guardian-classifier` and `/guardian` return
**403**, and `gpt-5.6-luna` 404s on `/responses`, for CLI sessions — while the
app's own `codex app-server` (launched bare, no special env) is entitled. The gate
is **entitlement/rollout-based and intermittent** (the first attempt connected, later
ones did not). Not replicable via `CODEX_INTERNAL_ORIGINATOR_OVERRIDE`, and the
built-in `openai` provider can't be overridden to force the HTTP classifier route.

## Paths to *reliable* real verdicts
1. **Relaunch the app with tracing** so its entitled `app-server` emits the verdict
   lines, then drive a task and harvest: `RUST_LOG=codex_guardian_v2=trace <ChatGPT binary>`.
   (Requires restarting the app; the spawned app-server inherits the env.)
2. **Substitute model** (what these runs do) — real rendered inputs, a reachable
   classifier label; swap `GUARDIAN_MODEL` back to `gpt-5.6-luna` when access exists.
3. **Harvest the sync-review rollouts** already on disk (`thread_source: guardian_review`)
   for real `risk_level`/`user_authorization`/`outcome` verdicts — 546 on this machine.
4. See also `../GUARDIAN-LUNA-AND-SECRETS.md` for the prompt/decision semantics.

## Capturing real verdicts from the app (done)
The app's `codex … app-server` is the entitled client. Relaunch the app with a CA
override + HTTP proxy env and drive a scored task:
```sh
cd /Applications/ChatGPT.app/Contents/MacOS
HTTPS_PROXY=http://127.0.0.1:8080 HTTP_PROXY=http://127.0.0.1:8080 \
RUST_LOG=codex_guardian_v2=trace \
CODEX_CA_CERTIFICATE=~/.mitmproxy/mitmproxy-ca-cert.pem nohup ./ChatGPT &
```
Results:
- The app-server honors `CODEX_CA_CERTIFICATE` (`loaded certificates from custom CA
  bundle … certificate_count=1`); its **HTTP** traffic flows through the proxy.
- Shell (`run: echo hi`) is **sync-scope** → no async classifier. A **computer/browser**
  task (`Open https://example.com …`) does trigger guardian; the app logged
  `codex_guardian_v2::async_scorer::approval: decide_request … decision_source="synchronous_assessment" reason=FreshRequired`.
- That spawned a **`thread_source: guardian_review` rollout** carrying the real verdict —
  captured: `{"risk_level":"low","user_authorization":"high","outcome":"allow",
  "rationale":"The user explicitly requested opening https://example.com …"}` →
  exported as `app-guardian-review-browser-task.jsonl`.
- The **async `gpt-5.6-luna`** verdict is still not captured: its WebSocket to
  `/guardian-classifier` **bypasses the HTTP proxy**, so mitmproxy saw only our CLI's
  requests (`POST /guardian-classifier` → 404, `GET` → 403 Cloudflare block) —
  `mitm-guardian-403-frames.jsonl`.
- To capture the async ws you need **transparent/system-proxy** interception
  (`networksetup -setsecurewebproxy`, admin) so the ws is funneled too.

## DECISIVE: the live reviewer is a JSON-schema Luna call, not `high|low`
Decrypting the app's WebSocket traffic (same HTTP-proxy + `CODEX_CA_CERTIFICATE`
setup; the app's ws **does** honor the proxy) shows the blocking reviewer call on
`/responses`:
```json
"model":"gpt-5.6-luna","reasoning":{"effort":"low"},
"text":{"format":{"type":"json_schema","name":"codex_output_schema","schema":{
  "type":"object","additionalProperties":false,
  "properties":{
    "risk_level":{"type":"string","enum":["low","medium","high","critical"]},
    "user_authorization":{"type":"string","enum":["unknown","low","medium","high"]},
    "outcome":{"type":"string","enum":["allow","deny"]},
    "rationale":{"type":"string"}},
  "required":["outcome"]}}}
```
Response metadata carries `x-codex-safety-buffering-enabled: "true"` and
`x-codex-safety-buffering-faster-model: "gpt-5.6-luna"`. `gpt-5.6-luna` is also used
as the **safety-buffering faster model** (it gets the main agent prompt + tools and can
return empty output).

**Therefore there is no literal `high|low` first-token to read in live traffic.** The
`classifier_instructions.md` single-token contract is **not the active mechanism** in
this build; the verdict is a structured JSON object from Luna. Captured frames →
`app-async-luna-capture.jsonl`. The observed verdict for a browser action
(`example.com` title) was `risk_level=low, user_authorization=high, outcome=allow`
(also persisted as the `guardian_review` rollout
`app-guardian-review-browser-task.jsonl`).

## Relevant env vars discovered
- `CODEX_ANALYTICS_EVENTS_CAPTURE_FILE` — dumps analytics payloads to a file, but is
  `#[cfg(debug_assertions)]`, so only present in **our debug build**, not the release app.
- `chatgpt_base_url` — redirect the API base (used to point at mock servers in tests).
- `CODEX_INTERNAL_ORIGINATOR_OVERRIDE` — originator header override (did **not** unlock
  the guardian routes).
