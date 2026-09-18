# Rosalind — OpenAI's AI-for-science product (unreleased)

Rosalind ("Helix") is an **unreleased biology/science research product** with its
own onboarding track, enrollment flow, dedicated models, and a custom WebGL DNA
helix. Not in the docs or UI yet.

## What it is (verbatim UI copy)
- Title: **Rosalind** · subtitle: *"Perform scientific research with frontier
  reasoning models and scientific tools in one place."*
- **Reason across biology** — *"Explore molecules, proteins, genes, pathways, and
  disease biology in one reasoning flow."*
- **Work with scientific tools** — *"Connect model reasoning to approved tools,
  datasets, and repeatable workflows."*
- **Evaluate evidence** — *"Compare findings across papers, datasets, and
  scientific context."*
- Opt-out affordance: *"I'm not a scientist"* / **Get started**.

## Dedicated models (codename leak)
```js
LLt = new Set(["gpt-rosalind-preview", "gpt-rosalind-5-5", "heisenberg"])
ILt = /rosalind/i
```
So Rosalind ships with **`gpt-rosalind-preview`**, **`gpt-rosalind-5-5`**, and a
model codenamed **`heisenberg`**.

## Onboarding + enrollment
- A first-run onboarding step: `RosalindWelcome: "rosalind-welcome"` (sibling of
  `FinanceWelcome`, `TeenWelcome`, `WindowsSandboxSetup`), copy under
  `electron.onboarding.rosalindWelcome.*`.
- **Enrollment** (product analytics `CODEX_ROSALIND_UX_*`):
  `ENROLLMENT_STARTED/SUCCEEDED/FAILED/NOT_REQUIRED`, `ANNOUNCEMENT_PRESENTED/
  ACCEPTED/DISMISSED/LEARN_MORE_CLICKED`, `PRIMARY_CTA_CLICKED`,
  entry sources `ACTIVATION_DEEPLINK`, `HOME_BEACON`, `ONBOARDING_COMPLETION`.
- Announcement key `chatgpt-rosalind-announcement-v1`; deeplink handler
  `show-chatgpt-rosalind-announcement` (source `activation_deeplink`).
- Plugin surface: query `["...","rosalind-plugin-info", …]`; announcement version
  gated by app version (`>= 0.146.0-alpha.7`).

## The "Helix" hero (custom 3D)
`rosalind-helix-*.js` (~33 KB) is a **WebGL2 particle DNA double-helix**:
attributes `aTangentDirection`, `aStrandPhase`, `aScale`, `aLayerOpacity`;
concepts `helix`, `backbone`, `particle`; `Float32Array` buffers; handles
`visibilitychange`, `pointermove`, `pointerleave`, `mobile`, `low-power`,
`hidden`, `data-theme`. Hero art: `chatgpt-rosalind-hero-*.png`.

## Access / gating
- `enrollment` required (enrollment-not-required path exists for some users).
- An internal check `isOpenAI(email)` (`endsWith("@openai.com")`) sits next to
  the Rosalind code — suggesting internal/limited rollout.

## Why it's juicy
It's a **whole unreleased product line** — an AI-for-science (biology) research
surface — with **dedicated frontier models** (`gpt-rosalind-preview`,
`gpt-rosalind-5-5`, `heisenberg`), a research onboarding/enrollment funnel, and a
custom 3D DNA-helix identity — none of it public yet.
