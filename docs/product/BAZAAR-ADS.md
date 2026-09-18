# "Bazaar" = ChatGPT's ads & personalization system

The codename **bazaar** in the desktop app is the internal name for **ChatGPT's
ads system** — a full ads control surface, not a store. It ships in the release
bundle with a complete Settings UI and backend endpoints.

## Endpoints
```
GET    /bazaar/history      → paginated ads history (cursor)
GET    /bazaar/preferences  → personalized ad topics
DELETE /bazaar/profile      → delete ads profile
PATCH  /settings/account_user_setting { feature: "bazaar_personalization_enabled" | "free_ads_opt_out", value }
```
Query keys: `["chatgpt-ads", <accountId>, "history"]`, `["chatgpt-ads", <accountId>, "topics"]`.
Account scoping via header `ChatGPT-Account-ID`.

## Account settings flags (from the account settings schema)
```
bazaar_personalization_enabled
bazaar_personalization_unavailable   (region gate)
free_ads_opt_out
```
Plus profile fields used for ad targeting: `accessory_id`, `birthday`, `chat_theme`.

## The UX model (verbatim copy, `ADS-SETTINGS-COPY.txt`)
- **Ads as a tradeoff:** *"Ads allow for more messages with ChatGPT."*
- **Three ways to go ad-free:**
  *"Upgrade to Plus"* / *"Change plan to go ad-free"* / *"you can reduce message limits to remove ads for free."*
- **"Ads are off"** state: *"You're using ChatGPT ad-free with reduced usage. Expand your access by upgrading your plan or turning on ads."*
- **Personalization:** *"Allow ChatGPT to use your past chats, activity, and preferences to select ads. Ads may still be based on your current chat."*
- **Topics:** *"Topics inspired by your activity"* — *"ChatGPT may use these topics to choose ads that are more relevant to you."*
- **History:** shows ads you *"viewed or interacted with"*, items as `{brand} — {title}`, with `Viewed`/`Opened`.
- **Region gate:** *"Personalized ads currently aren't available in your region. You'll still see ads based on limited data like general location, context, and device type."*
- **Delete ads data:** separate delete of *ads history* and *ads topics* (doesn't touch chats).

## Settings navigation
Section id: `settings.nav.ads-controls` ("Ads controls"). Sub-pages: History,
Topics, Personalization, Delete ads data, Upgrade options.

## Why it matters
The ChatGPT **desktop** app contains a complete, shipping **ads** subsystem —
including personalization explicitly derived from *"past chats, activity, and
preferences"*, an ads topic profile, an interaction history, a region-gated
personalization mode, and a tradeoff between ads and message limits. This is
enabled/rolled out via `bazaar_personalization_enabled` / `free_ads_opt_out` and
the `bazaar_personalization_unavailable` flag.

## Enabling / inspecting (dev mode)
- Statsig: `__STATSIG__.firstInstance.checkGate("bazaar_personalization_enabled")`
- Debug panel → `feature-access`; Query Devtools → `chatgpt-ads` keys.
- Delete ads data is exposed via the Settings → Ads controls page (or
  `DELETE /bazaar/profile`).
