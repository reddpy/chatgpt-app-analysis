# Experian credit-score flow (ChatGPT Personal Finance)

End-to-end trace of the credit-score connection inside **ChatGPT Personal
Finance** (`chatgpt_finance/src/FinanceCreditVerificationModels`,
`FinanceApi`).

## State machines
```
FinanceCreditVerificationConnectionStatus: connected | pending | failed | expired | requires_action
FinanceCreditVerificationStatus:           not_connected | pending | connected | requires_action | unknown
FinanceCreditVerificationEnvironment:      SANDBOX | PRODUCTION
```

## Flow
1. **Eligibility** — `GET /aip/ledger/credit/connection`
   → `decodeFinanceCreditVerificationEligibility` (`{ retryable, pending_deletion }`).
   Query key `["credit-connection"]`, `refetchOnMount:"always"`, `retry:false`.
2. **Start** — `POST /aip/ledger/credit/verification/start`
   → `decodeFinanceCreditVerificationSession` (contains a non-blank
   `verification_id`). This launches the hosted identity/credit verification
   (browser handoff; identity verification via Persona / vaulting via Very Good
   Vault — both are CSP-allowlisted hosts: `*.withpersona*.com`,
   `*.verygoodvault.com`).
3. **User completes** the external verification (status → `requires_action` /
   `pending` → `connected`).
4. **Complete** — `POST /aip/ledger/credit/verification/complete`
   `{ verification_id }` → `decodeFinanceCreditVerificationResult`
   (validates the id is non-empty).
5. **Score** — `GET /aip/ledger/credit/score`
   → `decodeFinanceCreditScoreSnapshot`.
6. **Disconnect** — `DELETE /aip/ledger/credit/connection`
   (and `POST /aip/ledger/credit/connection` to connect).

## Score snapshot data model
Fields: `score` / `score_value`, `score_date`, `updated_at`, `provider`,
`bureau`, `model` / `score_model`, `range`, `factors`, plus report metrics
`inquiry_count`, `available_credit`, `credit_age_months`, and utilization
(`finance_accounts_credit_percentage`). UI exposes: **"Credit checks"**
(inquiry count), available credit, credit age (months/years), utilization
`.
Provider is **Experian** (2,210 refs) using FICO-style scoring (`fico`, 186 refs).

## UI (ChatGPT Personal Finance)
- Product strings: `personalFinance.page.title`, `personalFinance.tabs.*`,
  `personalFinance.accounts.*` (connect/refresh/reconnect/remove, hide/show
  values), `personalFinance.accountDetail.*`.
- **Credit**: `finance_accounts_credit_*` — "Credit checks", "1 inquiry",
  utilization percentage, "score unavailable".
- Legal disclaimer shipped in the UI: *"…is not a licensed investment advisor
  and is not authorized to prepare tax returns."*

## Related
- Identifiers: "ChatGPT Personal Finance" connector name.
- Account linking (banks) is separate — see `AIP-LEDGER-FINANCE.md`
  (Plaid flow). Credit (Experian) is its own connection.
- Coin the whole surface is behind the `sheep` finance gateway flag
  (`chatgpt-ledger-widgets-sheep-fastapi`).

## Why it's notable
This is a **consumer credit product** in the shipping client: eligibility →
hosted identity/credit verification (Persona) → pull an **Experian** credit
snapshot (score, bureau, model, factors, inquiries, utilization, credit age) →
render it in a finance dashboard, with sandbox/production environments.
