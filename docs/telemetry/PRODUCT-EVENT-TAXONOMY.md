# Product event taxonomy — roadmap & feature intel

Extracted from the app's `logProductEvent` analytics enums (`CODEX_*`), 1,466
values total (raw list: `PRODUCT-EVENT-TAXONOMY.txt`). These are the events/fields
the desktop app reports to OpenAI's analytics. They double as a **feature/roadmap
map** — you can read unreleased product surfaces straight out of the enum names.

## New codenames / unreleased products

- **ROSALIND** — a feature with an **enrollment** flow:
  `ENROLLMENT_STARTED/SUCCEEDED/FAILED/NOT_REQUIRED`,
  `ANNOUNCEMENT_PRESENTED/ACCEPTED/DISMISSED/LEARN_MORE_CLICKED`,
  entry sources `ACTIVATION_DEEPLINK`, `HOME_BEACON`, `ONBOARDING_COMPLETION`.
  (Asset `chatgpt-rosalind-hero` exists → new enrollment/opt-in product.)
- **SKETCH** — a drawing/sketch surface with a full upload pipeline:
  `UPLOAD_STEP_BLOB_UPLOAD → CREATE_FILE_ENTRY → MARK_UPLOADED`,
  `INTERACTION_ACTION_OPENED/REMOVED/REOPENED/SAVE_COMPLETED`.
- **REPLAY** — replay onboarding (`LAUNCH_OUTCOME_*`, `FIRST_TURN_OUTCOME_*`).
- **MINI** — mini avatar/pet UI: `APPEARANCE_BAR`, `CUSTOM_PET`, `DEFAULT_PET`,
  `VISIBILITY_COLLAPSED/EXPANDED/EXPANDED_WITH_ACTIVITY_PILLS`,
  `QUICK_CHAT_OPEN_METHOD_CLICK/SHORTCUT`.

## Unreleased / partially hidden features

- **REFERRAL** — invite program (`INVITE_MODAL_ACTION_SEND_CLICKED`,
  `REFERRAL_ALREADY_EXISTS`, backend error taxonomy).
- **PROFILE** — profile edit + **social sharing to `LINKEDIN` / `REDDIT` / `X`**.
- **LEARNING** — "Learning blocks" (impressions, follow-ups, feedback,
  thumbnails).
- **GENERATED_IMAGE** — image editing: modes `COMMENT/REMOVE/RESIZE/SELECT`;
  entry points canvas/gallery/lightbox.
- **INLINE_VISUALIZATION** — trigger types `EXPLICIT`, `PASSIVE`, `SOFT`;
  sandbox failure reasons.
- **CONVERSATIONAL_ONBOARDING** — onboarding that *does real tasks* via
  computer-use: `HOLD_NEXT_FREE_HOUR`, `SEND_MESSAGE_TO_SELF`, `DESKTOP_NOTE`,
  `CSV_CHART`; access types CALENDAR_APP / MESSAGING_APP / COMPUTER_USE / DESKTOP.
- **APPGEN** — app generation: `CREATE_SITE_CLICKED`, `EDIT_WITH_CHAT_CLICKED`,
  pages ANALYTICS / DATABASE / SETTINGS; sources AT_MENTION / LIBRARY / SITES /
  RESPONSE_CARD / SITE_DETAILS.
- **AUTOMATION** — capability origins `FIRST_PARTY / MARKETPLACE / CUSTOM`,
  types `CONNECTOR / PLUGIN / SKILL`; actions created/deleted/run-now/updated.
- **GOOGLE_WORKSPACE** — artifact flows `DIRECT_CREATE / EDIT /
  LOCAL_ROUNDTRIP_EDIT`; resource kinds DOCUMENT / PRESENTATION.
- **REMOTE** — SSH connections + `REMOTE_CONTROL_ENROLLMENT`.
- **PRIMARY_RUNTIME** — bundled runtime (NODE, PYTHON, NODE_MODULES; releases
  LATEST / LATEST_ALPHA).
- **WINDOWS_SANDBOX** — readiness/setup stages.

## Safety / moderation surfaces (as tracked)
- **SAFETY**: protection types `BIO` / `CYBER`; surfaces `BLOCK` / `BUFFERING`;
  actions `FASTER_MODEL_CLICKED`, `TRUSTED_ACCESS_CLICKED`, `RETRY_*`.
- **CHATGPT_HANDOFF**: `ACCEPTED / REJECTED / TOOL_CALLED / UNDO`.
- **COMPUTER_USE_IPC**: transports `APPLE_EVENT`(s), client types
  `LEGACY_MCP / NATIVE_BRIDGE / NODE_REPL`, authorization failures (untrusted
  parent, relay without…).

## Infra / runtime
- **REALTIME** (162 events — largest family): voice media setup, device
  selection, permission states, error taxonomy.
- **PLUGINS** (115) / **BROWSER** (81) / **ARTIFACT** (51) / **DICTATION** (32) /
  **THREAD** / **PROJECT** / **FILE**.

## Why this is the juicy one
It's the closest thing to an **internal product roadmap shipped in the binary**:
every enumerated feature, its UI states, error taxonomy, and telemetry
dimensions — including surfaces not yet in the UI (ROSALIND enrollment, SKETCH,
REFERRAL, LEARNING, conversational computer-use onboarding, appgen sites).

Caveat: these are analytics enums (feature vocabulary), not server logic — they
reveal *what exists and is measured*, not how it's implemented.
