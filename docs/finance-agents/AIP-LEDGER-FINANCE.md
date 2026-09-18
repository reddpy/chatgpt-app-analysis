# AIP / Ledger = ChatGPT "Personal Finance"

`/aip/*` is the **AI Platform** namespace. `/aip/ledger/*` is the
**personal-finance** product; `/aip/connectors/*` is the connector platform;
`/aip/olympic/*` is a connector app (`connectors://…`, `internal://olympic`).

## Personal Finance surface
- Route **`/finances`** + sidebar item (`sidebar-finances-icon`).
- Query key `["personal-finance", <account>, "financial-memories"]`.
- **Bank linking via Plaid**: `plaid-link-*.js`, `plaid-callback-page-*.js`,
  CSP `cdn.plaid.com/link/v2/stable/link-initialize.js`, plus
  `https://production.plaid.com` / `sandbox.plaid.com`.
- **Identity verification via Persona** (`withpersona`, `Persona`).
- **Very Good Vault** (`verygoodvault`) — credential vaulting.
- **Credit**: `/aip/ledger/credit/score`, `/credit/connection`,
  `/credit/verification/{start,complete}`.

## Endpoints (`/aip/ledger/*`)
```
GET    /aip/ledger/profile
GET    /aip/ledger/institutions
GET/POST/DELETE /aip/ledger/links[/{link_id}]
GET/PATCH/DELETE /aip/ledger/links/{link_id}/accounts/{account_id}
POST   /aip/ledger/links/{link_id}/sync
GET    /aip/ledger/financial_memories ; DELETE .../{id}
GET    /aip/ledger/files ; GET .../{file_id}/download_link
GET/PATCH /aip/ledger/widgets ; /widgets/metadata ; /widget_preferences
GET    /aip/ledger/chats
GET    /aip/ledger/credit/{connection,score}
POST   /aip/ledger/credit/verification/{start,complete}
GET    /aip/ledger/oauth/state/{id} ; POST /oauth/start ; POST /oauth/submit-public-token
```

## Data model (fields seen)
`ledger_link`, `ledger_account`, `institution_id`, `institution_name`,
`account_name`, `account_type`, `institution_and_last_four` (e.g. "Chase ••••1234"),
`widget_type`/`widgetType`, `financial_memory`.

## Connectors (`/aip/connectors/*`)
A generic connector platform with OAuth + service accounts:
```
GET  /aip/connectors/{id} ; /{id}/actions ; /{id}/link ; /{id}/logo ; /{id}/tos
GET/POST/DELETE /aip/connectors/links[/{link_id}] ; .../list_accessible
POST /aip/connectors/links/oauth[/callback|/complete|/reauth] ; noauth
POST /aip/connectors/service_accounts/links/{oauth,noauth}
POST /aip/connectors/email/send_email ; /send_email_status ; /unsend_email
POST /aip/connectors/github/has_installations
POST /aip/connectors/google_contacts/search_contacts
GET  /aip/connectors/product_specific
GET  /aip/first-party/eligibility
```
So connectors can **send/unsend email**, query GitHub installs, search Google
contacts — agent-callable.

## Olympic (connector app)
`connectors://<on>` with `template: internal://olympic`; endpoints
`/aip/olympic/chats`, `/aip/olympic/oauth/callback`,
`/aip/olympic/debug/test-account-connection` (debug left in).

## Finance prompt templates (in-app skills)
- *"Prep a finance review"* — uses Google Calendar/Drive/Gmail/uploaded docs.
- *"Triage finance asks"* — Gmail/Slack/notes.
- *"Review a model"* — Drive/spreadsheet review of forecast/budget.

## Why it's juicy
ChatGPT ships a **personal-finance product**: connect real bank accounts (Plaid),
verify identity (Persona), pull credit scores, extract **financial memories**,
render ledger widgets, and run finance workflows (forecasts/budgets/reviews) —
all wired into the desktop client, gated server-side.
