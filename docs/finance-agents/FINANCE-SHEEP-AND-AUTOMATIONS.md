# Finance "sheep" flow & automations/RRULE engine

Two deep dives: (1) the personal-finance account-linking flow and its `sheep`
backend, (2) the automations/RRULE engine that powers scheduled tasks (and
hermes agent schedules).

---

# 1. Personal Finance — the "sheep" flow

`sheep` is the **finance backend gateway** (client `FinanceSheepApiClient`,
header `x-openai-use-sheep: 1`). It's gated by the experiment flag
**`chatgpt-ledger-widgets-sheep-fastapi`**:
```js
useSheep() = experimentService.isFeatureFlagEnabled({ flagName: "chatgpt-ledger-widgets-sheep-fastapi" })
```
The API has a **dual backend**: `useSheep() ? sheepClient.* : networkClient.*`
(new FastAPI "sheep" service vs the legacy ledger API).

## Client (`chatgpt_finance/src/FinanceApi`)
Methods: `userProfile`, `accountLinks`, `listInstitutions`, `startAccountLinking`,
`accountLinkingState`, `submitPlaidPublicToken`, `creditScore`,
`creditVerificationEligibility`, `startCreditVerification`, `removeCreditAccount`,
plus dashboard/widgets.

Decoders (schemas): `decodeFinanceUserProfile`, `decodeFinanceAccountLinks`,
`decodeFinanceAccountLinkingSession`, `decodeFinanceAccountLinkingStatus`,
`decodeFinanceInstitutionsPage`, `decodeFinanceCreditScoreSnapshot`,
`decodeFinanceCreditVerification{Eligibility,Result,Session}`,
`decodeFinanceDashboardPage`.

Enums: `FinanceDashboardPlacement` / `FinanceWidgetSpan` = `full | half | unknown`;
`FinanceStarterActionKind`; `financeWidgetInitialStateJson`,
`mergeFinanceDashboardPages`.

## Account-linking (Plaid) flow — end to end
1. `POST /aip/ledger/oauth/start` → `decodeFinanceAccountLinkingSession`
   (returns `{ link_token }`).
2. Renderer opens **Plaid Link** (`plaid-link-*.js`):
   ```js
   Plaid.create({ token: linkToken, onSuccess, onExit }).open()
   ```
   (`link-initialize.js` from `cdn.plaid.com`, CSP-allowed; Plaid prod+sandbox).
3. On success Plaid returns a **`public_token`**.
4. `PATCH /aip/ledger/oauth/state/{oauth_state_id}` `{ status: "initializing" }`.
5. `POST /aip/ledger/oauth/submit-public-token` `{ link_token, public_token }`.
6. Poll `GET /aip/ledger/oauth/state/{oauth_state_id}` →
   `decodeFinanceAccountLinkingStatus` until linked.
7. `POST /aip/ledger/links/{ledger_link_id}/sync` → pull accounts/transactions.
8. Accounts appear as `ledger_account` under an `institution`
   (`institution_and_last_four`, `account_type`).
9. Finish page: `plaid-callback-page-*.js`.

Identity/verification: **Persona** (`withpersona`) and **Very Good Vault**;
credit via `/aip/ledger/credit/{score,connection,verification/*}`.

Widgets: `GET/PATCH /aip/ledger/widgets`, `/widgets/metadata`,
`/widget_preferences` (dashboard widgets = finance cards, full/half span).

---

# 2. Automations / RRULE engine

Powers **scheduled tasks** (`/automations/*`) and hermes agent schedules. UI:
`automations-page`, `automation-dialog`, `automation-frequency-section`,
`automation-side-panel-tab`, `automation-delete-confirmation-dialog`.

## Model
An automation = **name + prompt + model + reasoning + sandbox + schedule +
destination + notification policy**, optionally from a **plugin template**
(`pluginTemplateId`).

Two kinds:
- **heartbeat** — recurring run continues **in this thread**; minute-based
  intervals (e.g. `FREQ=MINUTELY;INTERVAL=30`).
- **cron** — each run starts a **new task**; hourly/weekly/monthly schedules.
  Default is heartbeat; cron only when the user explicitly wants a new task.

Destinations: the current **thread**, **ChatGPT**, workspace host, Slack (owner-only for agents).

## Schedule (RRULE)
Built with the bundled `rrule` library. The custom-schedule dialog supports
FREQ `MINUTELY/HOURLY/DAILY/WEEKLY/MONTHLY/YEARLY`, `INTERVAL`, `BYDAY`,
`BYHOUR`, `BYMINUTE`, day-of-month / month / weekdays selectors.

Validation (`invalidRule`): *"Choose a repeating schedule with one time per rule,
at least 15 minutes apart, without a year, end date, or numbered weekday."*
Guidance: do **not** emit `DTSTART`; interpret wall-clock times in the user's
locale and encode `BYDAY/BYHOUR/BYMINUTE` directly.

## API (`/automations/*`)
```
GET    /automations ; /automation/{id}
POST   /automations/save ; /automations/remove ; /automations/set_status
POST   /automations/generate_creation_prompt
POST   /automation/{id}/run ; /automation/{id}/webhook_batch/run_now
PATCH/DELETE /share/automation/{id}
```
Statuses: `ACTIVE`, `ARCHIVED`, `AVAILABLE`; run history with
archive/unarchive, mark-all-read, "run now", plugin-template reset, and
cloud scheduled-task retry.

## App-server events
`heartbeat-automation-thread-state-changed`, `automation/…` notifications; the
thread carries `safetyBuffering`/`heartbeat` state.

## How they connect
Hermes agent schedules (`/hermes/agent/{id}/triggers`) reuse this engine: the
agent-schedule editor validates the same RRULE and requires the agent to be
**published** and its apps connected before a trigger can be added.
