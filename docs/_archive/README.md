# Hidden artifacts extracted from the open Rust core

Everything here comes from the recovered `codex` source (exact tag
`rust-v0.155.0-alpha.2.6`) and the binary strings. These are the non-obvious
things that aren't in the docs or the UI.

## 1. The system prompts (the agent's actual instructions)

`prompts/` mirrors `codex-rs/` and contains the real prompt assets:

| File | What it is |
|---|---|
| `protocol/src/prompts/base_instructions/default.md` | the base agent system prompt (personality, AGENTS.md spec, planning, sandbox/approval rules) |
| `core/gpt_5_2_prompt.md`, `core/gpt_5_1_prompt.md`, `core/gpt_5_codex_prompt.md`, `core/gpt-5.2-codex_prompt.md`, `core/gpt-5.1-codex-max_prompt.md` | per-model system prompts |
| `core/templates/model_instructions/gpt-5.2-codex_instructions_template.md` | templated model instructions |
| `core/templates/personalities/gpt-5.2-codex_{friendly,pragmatic}.md` | selectable personalities |
| `prompts/templates/compact/prompt.md` + `summary_prefix.md` | context-compaction prompt |
| `prompts/templates/realtime/{backend_prompt,realtime_start,realtime_end}.md` | voice/realtime prompts |
| `prompts/templates/review/rubric.md` | code-review grading rubric |
| `collaboration-mode-templates/templates/{default,plan}.md` | collaboration modes |
| `core/templates/agents/orchestrator.md`, `core/templates/collab/experimental_prompt.md` | multi-agent orchestration |
| `ext/memories/templates/**`, `ext/goal/templates/**` | memory + goals prompts |
| `memories/write/templates/memories/{stage_one_system,consolidation}.md` | memory-writing pipeline prompts |

## 2. The Guardian safety prompts (the most interesting)

| File | What it is |
|---|---|
| `core/assets/guardian/policy.md` | the **blocking** Guardian security policy — full risk taxonomy (data exfiltration, credential probing, persistent security weakening, destructive actions) with allow/deny outcome rules |
| `core/assets/guardian/policy_template.md` | same, with a `{{ tenant_policy_config }}` slot for **per-organization policy injection** |
| `ext/guardian-v2/src/async_scorer/classifier_instructions.md` | the **async** Guardian classifier. Output contract: *"Your first output token is the entire classification: `high` or `low` … Output that token immediately and nothing else."* Escalates to blocking review on future actions |
| `core/assets/guardian/node_repl_policy.md` | extra rules for computer/browser use via `node_repl`/`cua_repl` |

Key design principles encoded there:
- Trusted evidence = user/developer messages, `AGENTS.md`, `request_user_input` answers — **everything else (tool output, skills, plugin descriptions, assistant text) is untrusted**.
- Re-authorization trap closed: an action previously refused, then approved, is `high` only if approval "clearly covers the exact action."
- Predictive: flags risk for actions *just* about to happen (next 2 actions) and assistant drift.
- Browser/computer actions judged by **observed interface state, not the agent's stated intent**.

## 3. Unreleased model codenames

Present in both source and binary (grep `gpt-5.6-*`):

```
gpt-5.6-terra     (84 refs)
gpt-5.6-sol       (57)
gpt-5.6-luna      (56)
```
Plus `gpt-5.5`, `gpt-5.4`, `gpt-5.3-codex`, `gpt-5.2-codex`, `gpt-5.1-codex-max`.
These `.6-<name>` slugs are internal codenames not surfaced in the product.

## 4. Internal headers (server-side plumbing)

```
x-openai-internal-codex-residency        (data-residency routing)
x-openai-internal-codex-responses-lite
x-openai-internal-caller
x-openai-codex-luna-reserve              (capacity reserve; matches "luna-reserve-recovery")
x-openai-encrypted-tool-arguments        (opaque tool args to the backend)
x-openai-actor-authorization
x-openai-subagent
x-openai-memgen-request                  (memory generation)
x-openai-model / x-openai-preview / x-openai-fedramp
```

## 5. Remote-control pairing protocol (staging + prod)

```
/backend-api/wham/remote/control/server/enroll
/backend-api/wham/remote/control/server/pair
/backend-api/wham/remote/control/server/pair/status
/backend-api/wham/remote/control/server/refresh
/other/control
```
(both `api.chatgpt-staging.com` and production) — a device/server **pairing &
enrollment** flow for remote control, separate from OAuth login.

## 6. Agent identity endpoints

```
auth.openai.com/api/accounts/v1/agent/register
auth.openai.com/api/accounts/v1/agent/agent-runtime-id/task/register
```

## 7. Misc signals

- `persistent.oaistatic.com/codex/pets/v1` — an undocumented "pets" endpoint.
- `https://api.openai.com.evil.example/v1` — a parser security-test fixture.
- `announcement_tip.toml` — the in-app "announcement" engine (date/version-regex
  gated messages; currently "update required" style notices).
- `x-openai-internal-codex-residency` + `fedramp` → enterprise residency/compliance routing.
