# Juicy findings pass 2 (dev-console era)

## 1. Embedded secret-detection / redaction engine (gitleaks-class)
The renderer ships a **secret scanner** that finds ~90 token families in text and
replaces them with `<SECRET>` before logging/telemetry. Full pattern list:
`SECRET-REDACTION-ENGINE.txt`.

Mechanism: `gq(/regex/g, prefilterLabels...)` builds a fast prefixed regex;
matches are merged (`ola`) and spliced out (`sla` → `<SECRET>`); there are also
entropy/hygiene checks (`nla`: length 16–4096, charset variety, not
`password/secret/token/none/null/true/false`, not `<3` unique chars, rejects
`example/dummy/placeholder/changeme/redacted/...`).

**Token families it recognizes** (the interesting part — includes internal-looking
formats):
```
ops_        ← OpenAI "ops" JWT-ish tokens (\bops_eyJ…{250,4096})
atlasv1     ← Atlas v1 tokens (14.b64.atlasv1.<60-70>)
hrku-aa     ← HRKU-AA…{58}
fo1_ / fm1 / fm2_   ← fo1_…, fm1a/fm1r_, fm2_…
a3- / a3t   ← A3-XXXXXX-... tokens
akcp, cmvmd, absk, pat, _mmk, q~, sha256~
dt0c01.     ← Auth0
```
Standard providers: AWS (AKIA/ASIA/ABIA/ACCA/A3T), GCP `AIza`, GitHub
(`ghp_/gho_/ghu_/ghs_/ghr_/github_pat_`), GitLab (`glpat-/gldt-/glptt-/glrt-`),
Slack (`xoxb-/xoxa-/xoxp-/xoxr-/xoxs-/xapp-/xoxe`), Stripe
(`sk_/rk_ test|live|prod`), SendGrid `SG.`, HuggingFace (`hf_`, `api_org_`),
PyPI, npm, OpenAI (`sk-proj-/sk-svcacct-/sk-admin-`, `sk-…T3BlbkFJ…`), Anthropic
`sk-ant-admin01/api03`, Databricks `dapi`, DigitalOcean `doo/dop/dor_v1_`,
Dropbox `dp.pt.`, Facebook `EAA[MC]`, Sentry (`sntryu_/sntrys_`), Shopify
(`shpat_/shpca_/shppa_/shpss_`), Slack, Linode `lin_api_`, PlanetScale, Pulumi
`pul-`, New Relic (`NRJS-/NRII-/NRAK-`), Notion `ntn_`, Perplexity `pplx-`,
PostHog `phx_`, Prefect `pnu_`, Rubygems, Cloudflare `v1.0-`, Twilio `SK…`,
Supabase (`hvb./hvs.`), Age keys, private keys (`-----BEGIN … PRIVATE KEY-----`),
and generic `TOKEN|SECRET|PASSWORD|API_KEY|COOKIE|_PAT` assignments.

## 2. New codenames (feature flags / settings)
- **atlas** — appears as a browser family alongside `chrome` (`['atlas','chrome']`) and as a token format (`atlasv1`).
- **tatertot** — an artwork/illustration theme set (`tatertot: {16:…, 20:…}`, `tatertot:['study']`), alongside `cme`, `picture_v2`, `search`.
- **m3m** — workspace feature: `m3mEnabled`, `m3mOptedOut`, `m3mWorkspaceEnabled` (`beta_settings.m3m_workspace_enabled`).
- **bazaar** — a store/marketplace with personalization + ads: `bazaar_personalization_enabled`, `bazaar_personalization_unavailable`, `free_ads_opt_out`, `accessory_id`, `birthday`.
(+ earlier: owl, aperitif, sky, walnut, wham, estuary, skysight, maitai, tinysky,
gaas, rosalind, luna/terra/sol, sheep.)

## 3. New feature gates (subset)
```
atlas_mode_enabled            lockdown_mode_enabled        m3m_workspace_enabled
is_tatertot_enabled           add_to_flashcards_enabled    instant_answers_enabled
bazaar_personalization_enabled assistant_revisions_enabled greeting_enabled
template_elicitation_enabled   realtime_memory_summary_enabled
realtime_continuity_enabled    continue_in_voice_enabled    thumbnail_previews_enabled
web_file_library_enabled       auto_top_up_enabled          is_custom_checkout_enabled
desktop_app_beta_enabled       product_carousel_see_more_enabled
```
Notable: **flashcards**, **lockdown mode**, **instant answers**, **auto top-up /
custom checkout** (billing), **birthday/accessory** (profile), **free_ads_opt_out**.

## 4. App-server event surface (core → desktop), 83 methods
Full list: `analysis/appserver-methods.txt`. Families: `thread/*` (29),
`item/*` (14), `turn/*`, `mcpServer/*`, `account/*`, `fs/*`, `hook/*`,
`fuzzyFileSearch/*`, `codex/event/*`, `codex/toolSurface`,
`item/autoApprovalReview/*`, `openai/outputTemplate`, `openai/resourceActivities`,
`remoteControl/status/changed`, `externalAgentConfig/import/*`,
`windowsSandbox/*`, `skills/changed`, `modelProvider/authRecovery*`,
`rawResponse(Item)/completed`, `process/{outputDelta,exited}`.

## 5. How to use these in dev mode
- Console: `__STATSIG__.firstInstance.checkGate("atlas_mode_enabled")`,
  `…overrideGate("instant_answers_enabled", true)`.
- Query Devtools: inspect cached backend data for the endpoints behind these.
- Debug panel → `feature-access` to see gate state.
