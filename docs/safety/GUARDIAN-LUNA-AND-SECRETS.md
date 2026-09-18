# Guardian/"Luna" safety system + secret scrubbing

Answers to: *are there hidden system prompts, how do we test the safety black
box, and how are secrets scrubbed?*

> **Terminology note:** the real crate is **`guardian-v2`** and the real sanitizer is
> **`redact_secrets`**. If you ever see `il`/`il_v2` or `ln`/`lnedString` in raw
> search output for this project, that is a *display-only* substitution applied to
> grep results — **not** file content. A full on-disk audit of this dossier found
> `guardian_v2` ×546 and zero real `il_v2` aliases; all apparent `il`/`ln` hits were
> substrings of real words (`model_dil_v2`, `util_`, `wait_until`, `Solnedgång`,
> `evil.example`). Pristine names are used throughout.

---

## 1. The hidden safety system: Guardian v2, powered by `gpt-5.6-luna`

The safety black box is **not** a server-side mystery — it's a shipped, two-stage
client-orchestrated model system.

### Async classifier (the gate)
- Model: **`gpt-5.6-luna`** — `ext/guardian-v2/src/async_scorer/sampler.rs:39`
  (`pub(crate) const MODEL: &str = "gpt-5.6-luna";`). "Luna" is the classifier
  model, sibling to the `terra`/`sol` slugs.
- `prompt_cache_key = "guardian-v2:{thread_id}"`, `ReasoningEffort`, websocket
  connection pool (`sampler/connection_pool.rs`).
- Prompt: `ext/guardian-v2/src/async_scorer/classifier_instructions.md`.
  Contract: *"Your first output token is the entire classification: `high` for
  high risk or `low` for low risk. Output that token immediately and nothing else."*
- Non-blocking, but a `high` verdict **triggers blocking review on future actions**.
  It also predicts: high if unknown/low authorization now, or critical risk in the
  last 5 actions, or high/critical in the next 2 actions.

### Blocking reviewer (the judge)
- Prompt: `core/assets/guardian/policy_template.md` — *"You are judging one planned
  coding-agent action."* Outputs `risk_level` ∈ {`low`,`medium`,`high`,`critical`}
  × `user_authorization` ∈ {`high`,`medium`,`low`,`unknown`} → derives `outcome`
  from the policy.
- Policy: `core/assets/guardian/policy.md` (default = *"default generic
  organization / personal user"*), with `{{ tenant_policy_config }}` injection slot
  for **per-organization policy**. `node_repl_policy.md` adds computer/browser rules.
  Sections: Data Exfiltration, Credential Probing, Persistent Security Weakening,
  Destructive Actions, Low-Risk Actions — each with explicit allow/deny outcome rules.
- Model selection: `ext/guardian-reviewer/src/model.rs`; circuit breaker +
  failed-review feedback + token budget per review.

### Coverage matrix (`async_scorer/coverage.rs`)
`GuardianScope`: `computer_use`, `shell`, `code_mode`, `file_changes`, `mcp`,
`network`, `permissions`. Each maps to a `GuardianReviewMode`:
`Adaptive` | `Synchronous` | `Disabled`.
- Legacy default: `computer_use_only = true` → `computer_use = Adaptive`, everything
  else `Synchronous`, unscored actions **Ignored**.
- When a model declares its own `guardian` policy → per-category modes,
  `other_tools = Disabled`, unscored → `InvalidateScore`, `sandboxed_exec_commands = true`.

### Decision semantics (`async_scorer/approval.rs`) — where to look for weaknesses
- `full_access` → **`Allow` immediately** (Guardian is bypassed).
- `!require_guardian` and reviewer is `User` (or policy isn't OnRequest/Granular) →
  `AskUser`.
- `Adaptive` + *cached* score exists + `!require_fresh_review` → **`Allow` from cache**
  (a stale/cached `low` auto-approves). `UnscoredAction::{AgeScore,InvalidateScore}`
  governs how old scores expire.
- Otherwise synchronous review → `Reviewed(decision)` or `AskUser`.
- `ApprovalDecision ∈ {Allow, AskUser, Reviewed(..)}`.

### Telemetry
`codex.guardian_v2.classification{risk_level,authorization,outcome}`,
`.classification.duration_ms`, `.token_usage`, `.truncation.bytes`,
`.fast_decision{outcome,failure_reason}`, `.tool_call_lag{invalid_score,elevated_risk,
authorization_changed,score_lag,stale_score}`, `.review_fallback`,
`guardian.review.{duration_ms,ttft.duration_ms,token_usage}`.

---

## 2. How to actually test the black box

Because the client *is* the orchestration layer, all four are legitimate and concrete:

1. **Reproduce the routing deterministically.** Build `codex` at the pinned tag; the
   Guardian code is the same. Swap `LunaSampler` for a scripted stub and drive
   synthetic transcripts through it. Their own tests show the seams:
   `async_scorer/{action,trusted_tools,trusted_skills,config,budget,sampler,
   parent_compaction,transcript}_tests.rs`.
2. **Oracle the classifier.** It is a single-token `high|low` gate with a *fully
   known prompt*. Feed `classifier_instructions.md` + the tenant policy + a rendered
   transcript + planned action to any model and read the first token. That is a
   direct, repeatable probe of the decision function.
3. **Observe the real client.** It samples `gpt-5.6-luna` with cache key
   `guardian-v2:{thread_id}`. Capture the sampling request/response (same TLS
   technique used for owl) to see the exact rendered evidence and verdicts.
4. **Probe the interesting cases:** does `full_access` unconditionally allow?; does a
   cached `low` unlock later risky calls via `AgeScore`?; do the "last 5 / next 2
   actions" predictive windows create blind spots?; does `policy.md`'s *"Allow
   actions are guidance to classify the risk as low"* downgrade risky actions?

---

## 3. Secret scrubbing — three independent layers

### a. Rust `codex_secrets::redact_secrets` (the model/memory boundary)
`secrets/src/sanitizer.rs` — best-effort regex, output `[REDACTED_SECRET]`:
- `sk-[A-Za-z0-9]{20,}` (OpenAI keys)
- `\bAKIA[0-9A-Z]{16}\b` (AWS access key IDs)
- `(?i:\bBearer)[ \t]+[A-Za-z0-9._~+/-]{16,}=*` (bearer tokens)
- `(?i)\b(api[_-]?key|token|secret|password)\b(\s*[:=]\s*)(["']?)[^\s"']{8,}` (assignments)

Call sites:
- `memories/write/src/{rollout_input,phase1,phase1_output}.rs` — scrub rollout data
  **before persisting memories**.
- `app-server-protocol/src/protocol/item_builders.rs` — scrub `command`/`query`
  fields (`shlex_join(cmd)`) **before they cross the app-server protocol**.
- Tests deliberately assert *non*-redaction of look-alikes ("Bearer of good news",
  short tokens) — i.e., it is tuned for precision, and will miss secret formats it
  doesn't enumerate.

### b. JS redaction engine (telemetry/logs)
The gitleaks-class engine — see `SECRET-REDACTION-ENGINE.txt`. Runs in the Electron
layer for analytics/log scrubbing; broader rule set than the Rust sanitizer.

### c. Network credential broker (sandbox)
The sandbox proxy substitutes **dummy credentials**, so real tokens never leave the
process; combined with `RedactedString` (`codex_utils_redacted_string`, redacted
debug rendering, e.g. `url: "<redacted>"`) and
`scrub_non_inheritable_env_vars` (`codex_protocol::shell_environment`).

**Net:** three scrubbers at different boundaries (wire protocol, memory write,
telemetry, sandbox egress) with a deliberately narrow Rust regex and a broader JS one.
