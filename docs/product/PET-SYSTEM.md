# GOLDEN FIND — "Codex Pets": a hidden Tamagotchi-style subsystem

This is the most hidden thing in the desktop bundle, and it is not a small
Easter egg — it's a full subsystem with assets, animation, a custom-pet pipeline,
a floating overlay, XP, and a world.

## The roster (built-in pets, each with its own spritesheet)

| Pet | Spritesheet | Size |
|---|---|---|
| `bsod` (blue screen of death) | `bsod-spritesheet-v5-…webp` | 1.36 MB |
| `codex` | `codex-spritesheet-v6-…webp` | 1.31 MB |
| `dewey` | `dewey-spritesheet-v5-…webp` | 1.16 MB |
| `fireball` | `fireball-spritesheet-v5-…webp` | 1.45 MB |
| `hoots` | `hoots-spritesheet-v8-…webp` | 2.45 MB |
| `null-signal` | `null-signal-spritesheet-v7-…webp` | 0.80 MB |
| `rocky` | `rocky-spritesheet-v5-…webp` | 0.98 MB |
| `seedy` | `seedy-spritesheet-v10-…webp` | 1.28 MB |
| `stacky` | `stacky-spritesheet-v6-…webp` | 1.05 MB |

`rocky` is the default (79 references). Spritesheets are versioned (v5–v10) and
support **directional frames** ("looking directions").

## Mechanics

- **XP** and **streak/usage stats**: `totalTextTokens`, `peakTokens`,
  `longestTaskDurationMs`, `currentStreakDays`, `longestStreakDays`
  — the pet is fed by your coding activity.
- **Floating overlay**: show/hide, resize ("Adjust the size of your pet"),
  restore default size, "bar-only Mini mode without a pet", keyboard shortcut,
  customization options.
- **Hatching**: there is a "Hatch Pet is disabled" state.
- **Fainting**: `fallen-pet-a467195bac78.png`.
- **Work Mode pet**: exported symbol `WorkModePetContent` (pet reacts to work).
- **World/garden**: `garden-expo`, `earth-travel-world.svg`, and
  `model-*.js` which is actually **3D terrain generation** (`single-hill`,
  `two-hills-and-saddle`, `valley-and-ridge`, contour marching squares).
- **Mini-games for the hardware**: `codex-micro-mini-games` ships
  `brick-breaker`, `asteroids`, `snake`.

## Custom pets (the interesting part)

- UI: *"Create your own pet"*, *"Create pet"*, *"Customize pet"*,
  *"Custom pets"*, *"Create a pet based on what you know about me"* — the
  **agent can generate a pet from your data**.
- Pipeline: *"open the local custom pet folder"*, *"refresh custom pets from
  local manifests"*, *"updating an outdated custom pet"*.
- Pet IDs can be `uploaded:pet_…`; server pets come with
  `spritesheet_url` + `spritesheet_url_expires_at` (URLs expire, cached ~8 min
  with a 480 s staleness check).
- **Manifest schema** (from the code): `petName`, `directions`, `rows`,
  `columns`, `frameWidth`, `frameHeight`, `spritesheet_url`.

## Backend
Endpoint family `codex/pets/v1` (`https://persistent.oaistatic.com/codex/pets/v1`)
plus a `pets-settings-route` and the flag-gated Pets settings page.

## Files (prettified) in `feature-code/`
`pet-model-*.pretty.js`, `codex-pet-assets-*.pretty.js`,
`pet-content-*.pretty.js`, `pets-settings-route-*.pretty.js`,
`pet-install-state-*.pretty.js`, `garden-expo-*.pretty.js`,
`puppet-*.pretty.js`, `model-catalog.pretty.js` (terrain generator).

## Why this is the "golden" find
It's a complete, unreleased **gamified companion product** — sprites, animation,
activity-driven progression, user-generated content (custom/agent-generated
pets), a world, and hardware mini-games — shipped silently inside the bundle and
gated behind flags/settings. Nothing about it appears in the product UI or docs.
