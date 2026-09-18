# Plaid integration & ledger widgets schema

Continuation of `AIP-LEDGER-FINANCE.md` / `EXPERIAN-CREDIT-TRACE.md`.

## 1. Plaid Link integration (`plaid-link-*.js`)

**Script loading**
- Injects `<script src="https://cdn.plaid.com/link/v2/stable/link-initialize.js">`
  with `async`, `referrerPolicy="no-referrer"`, `data-codex-plaid-link="true"`,
  and the page's CSP `nonce`.
- Reuses `window.Plaid` if already present; load timeout → *"The financial
  account connection took too long to load"*.

**Opening Link**
```js
Plaid.create({
  noLoadingState: true,
  receivedRedirectUri,          // for OAuth resume
  token: linkToken,
  onLoad: () => s.open(),
  onSuccess: (publicToken) => …,
  onExit: (err) => err == null ? cancelled : failed,
})
```

**Flow orchestration**
1. `startOAuthFlow({ link_id, native_android:false, native_ios:false })`
   → `{ token (link_token), oauth_state_id }`.
2. Open Plaid Link with the token.
3. On success → `submitPlaidPublicToken({ link_token, public_token })`.
4. `markOAuthFlowInitializing(oauth_state_id)`.
5. **Poll** `getOAuthState(oauth_state_id)` (every 2 s, up to a timeout) until
   `finished === true` or `connected_accounts > 0`; throw on
   `status === "error" | "disconnected"` (*"The financial account connection was
   declined"*).
6. Invalidate queries `["personal-finance", accountId]` and the accounts query.

**OAuth redirect resume** — many banks redirect; the pending flow is persisted in
`sessionStorage` under **`codex:finance:pending-plaid-link`**
`{ accountId, createdAt, linkToken, oauthStateId, returnTo }`, and resumed via
`receivedRedirectUri` + `oauth_state_id`. Default `returnTo` = `/finances`.

**Guards / UX**
- Only **one** connection flow at a time (`"A financial account connection is
  already in progress"`).
- Plaid's `window_open` message (popup) is intercepted and opened in the
  **external browser** (`open_in_browser_bridge`), then `stopImmediatePropagation`.
- `AbortSignal` wiring throughout (component unmount cancels the flow).

## 2. Two linking backends: Plaid **and Apple FinanceKit**
The account model includes **`is_financekit_link`** — so accounts can come from
**Apple FinanceKit** (on-device financial data) as well as Plaid.

## 3. Account / ledger model
```
connected_accounts · linked_accounts · manual_accounts
persistent_account_id · display_account_type · display_balance
net_balance · total_balances · list_transactions
```
Modules: `FinanceModels`, `FinanceInstitutionModels`,
`FinanceAccountLinkingModels`, `FinanceCreditScoreModels`,
`FinanceCreditVerificationModels`, `FinanceProfileModels`, `FinanceApi`.

## 4. Dashboard widgets schema
- **`include_dashboard`** flag per widget; onboarding widget id
  `ledger_finance_onboarding_widget`.
- **Widget span/placement**: `full | half | unknown` (`FinanceWidgetSpan`,
  `FinanceDashboardPlacement`); `FinanceStarterActionKind`; state serialized via
  `financeWidgetInitialStateJson`; page-merging `mergeFinanceDashboardPages`.
- **Dashboard settings** (user-editable): choose + **reorder** widgets
  (`moveUp`/`moveDown`), states **Hidden / Always shown / Position fixed**, reset,
  save; plus **category editing** (*"Could not save category edits"*).
- Empty state: *"Your dashboard will populate as your financial data becomes
  available."*
- Widget data types seen: `net_balance`, `total_balances`, `credit_score`,
  `list_transactions`, per-account `display_balance`.

## Why it's notable
The finance stack is **multi-source by design** — **Plaid** (bank OAuth, incl.
redirect resume), **Apple FinanceKit** (on-device), **Experian/FICO** (credit),
**Persona** (identity), **Very Good Vault** (secrets) — feeding a configurable
widget dashboard, all behind the `sheep` gateway.
