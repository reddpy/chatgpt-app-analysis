# Credits, usage limits & billing (the third thread)

While tracing finance/automations, the credits/billing layer surfaced.

## Usage windows
Codex usage is metered in two windows (`codex.rateLimitResetPromptModal.*`):
- **5 hour usage limit** (`windowMinutes` ≈ 300)
- **Weekly usage limit** (≈ 10080)
Each window has `remainingPercent` + `resetsAt`.

## Usage resets ("banked resets")
- Users can hold **reset credits** (`status === "available"`), shown as
  "{count} available"; each has `id`, `title`, `expires_at`.
- Actions: redeem one reset, **"Pay {price} to reset"**, **"Upgrade plan"**,
  **"Add credits"**, or wait until the weekly reset date.
- Copy: *"Pay {price} to reset your usage limits to 100% immediately or wait
  until {resetDate}…"*; *"Use your banked reset, or save it for later and pay
  {price}…"*.
- Telemetry: banner events `codex_usage_limit_banner_shown`,
  `codex_usage_limit_banner_cta_clicked` posted to
  **`POST /wham/analytics-events/events`** (`platform: codex_desktop_app`).

## Credits + auto-reload (billing)
- **Credits** with quantity tiers; validation rules (min 125, increments of 250,
  `credit_purchase_quantity_max` cap 250,000).
- **Volume discounts** (`volume_discount_v1` tiers: 2500→10%, 5000→20%,
  25000→30%; `volume_discount_with_auto_reload_incentive_v1` adds +10%).
- **Auto-reload / auto top-up** (`auto_top_up_enabled`): set a threshold / target
  balance / monthly limit and a payment method (`recharge_threshold`,
  `recharge_target`, `recharge_monthly_limit`, `payment_method_id`).
- Dialogs: `CreditReloadDialogHost`, combined vs legacy purchase/auto-reload
  intents.

## Checkout
```
GET  /subscriptions/credits/discount-offer
POST /payments/checkout  { checkout_source:"codex-embedded-checkout",
                           checkpoint_ui_mode:"custom"|"redirect",
                           entry_point:"credit_purchase_modal",
                           credit_purchase_data:{ account_id, quantity, unit:"credit", discount_offer_id },
                           cancel_url }
```
Response: either a `custom_checkout_session` (embedded, opened in-app) or a
`url` (external browser). Business owners get an **embedded credit checkout**
gated by `use_embedded_credit_checkout` / `enable_embedded_credit_checkout`
(Statsig dynamic config `4062279831` / `1721641661`).

## Related gates
`1721641661` (`skip_native_modal_*`), `931825599` (`enabled`),
`4062279831` (`enable_embedded_credit_checkout`), `1112993408`
(`use_embedded_credit_checkout`), plus the `*_top_up_*` / `custom_checkout`
settings flags.

## Why it matters
It completes the monetization picture alongside **Bazaar ads**: usage-metered
limits (5h + weekly), consumable **resets**, purchasable **credits** with volume
discounts + **auto-reload**, and an **embedded Stripe checkout** inside the app —
with its own analytics endpoint (`/wham/analytics-events/events`).
