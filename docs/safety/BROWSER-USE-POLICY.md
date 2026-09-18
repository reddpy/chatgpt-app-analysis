# The Browser Use policy (app-only, fully recovered)

Recovered from the bundled service
`Resources/plugins/openai-bundled/plugins/{browser,chrome}/scripts/browser-service.mjs`.
It is a two-part policy: a **per-origin policy** and a **network policy**,
sourced from two places that are merged with requirements/enterprise taking
priority:

- `readAll()` → `config.browser_use` (user/local config)
- `readRequirements()` → `requirements.browserUse` / `requirements.network`
  (managed/MDM-pushed)

## 1. Per-origin policy

Each origin entry (and a `defaultOriginPolicy`) can set:

| Field | Values | Default |
|---|---|---|
| `access` | `allow` / `deny` | `allow` |
| `uploads` | `allow` / `deny` | `allow` |
| `downloads` | `allow` / `deny` | `allow` |
| `fullCdpAccess` | `allow` / `deny` | `allow` |
| `autoReview` | `allow` / `deny` | `allow` |
| `persistentApproval` | bool | `true` |
| `accessApprovalLifetime` | `turn` / `thread` | `thread` |

**Hard rule (from the code):**
```js
if (policy.access === "deny")
  policy = { ...policy, uploads:"deny", downloads:"deny",
             fullCdpAccess:"deny", autoReview:"deny" };
```
So denying `access` to an origin cascades to deny uploads, downloads, raw CDP,
and auto-review.

Merge precedence: per-origin > `defaultOriginPolicy`; requirements override config.

### Origin pattern syntax (`dx` / `ax`)
| Pattern | Meaning |
|---|---|
| `*` | any host |
| `**.example.com` | apex **and** all subdomains |
| `*.example.com` | subdomains only (not apex) |
| `example.com` | exact host |

Rules enforced: must be `http://` or `https://`; scheme **and port** must match;
leading/trailing whitespace or embedded `\s` invalidates the entry; `%2a` is
normalized to `*`; IPv6 hosts use `[brackets]`.

## 2. Network policy (domain allow/deny)

Fields: `enabled`, `allowedDomains`, `deniedDomains`, `hardDenyAllowlistMisses`
(from `managedAllowedDomainsOnly`), plus `default_permissions` →
`permissions[profile].network`.

Decision logic:
```js
if (network.enabled === false) return "deny";
if (domainIn(deniedDomains))    return "deny";
if (!network.hardDenyAllowlistMisses || domainIn(allowedDomains)) return null;
return "deny";
```
`hardDenyAllowlistMisses` = **allowlist-only mode** (managed): anything not in
`allowedDomains` is denied. Domain matching supports `*`, `**.x` (apex+subs),
`*.x` (subs).

## 3. Enforcement path

- `ensureUrlPolicyAllowed(url, check)` runs on navigation/download/general
  operations.
- `ensureRawCdpUrlAllowedForOperation(url, "raw-cdp-destination-url")`:
  - if `fullCdpAccess` is `deny` → audit with reason **`enterprise_policy_blocked`**
    and message *"…cannot use raw CDP because the Browser Use origin policy blocks it."*
  - if unavailable → reason **`enterprise_policy_unavailable`**, message
    *"…could not verify the Browser Use origin policy before using raw CDP."*
  - otherwise falls through to an elicitation (user prompt) unless persistent
    approval applies.
- Persistent approval scope key: `browser_use_persistent_approval_scope`,
  value `all-sites`; gated by `allowGlobalPersistentApproval`.

## 4. Security modes (`BROWSER_USE_SECURITY_MODE`)
```
""                          => Default
"disabled-for-local-testing" => DisabledForLocalTesting
"gaas-browser-environment"  => GaasBrowserEnvironment
```
`gaas-browser-environment` requires the automated-safety-prechecks flag
(`BROWSER_USE_AUTOMATED_SAFETY_PRECHECKS_ENABLED === "1"`) —

```js
securityMode === "gaas-browser-environment" && safetyPrechecksEnabled
```

## 5. Credential-field protection (enforced at clipboard layer)
```js
attributes: ["type","autocomplete","id","name","placeholder","aria-label","title"]
pattern: /user[-_ ]?name|e[-_ ]?mail|one[-_ ]?time[-_ ]?code|password|
          passcode|passwd|\botp\b|\b(?:2fa|mfa)\b|phone|mobile|\btel\b/i
```
Applies to cut/copy/paste via the virtual clipboard; hidden input `value`
attributes are stripped during page-content export.

## Summary
The policy is an **enterprise-controllable, per-origin permission model** with
four independent capability switches (`uploads`, `downloads`, `fullCdpAccess`,
`autoReview`) plus an `access` master that cascades, an approval-lifetime model
(`turn`/`thread`), a separate **network domain allow/deny** layer with a
managed allowlist-only mode, wildcard semantics (`*`, `*.`, `**.`), and three
security modes. Managed (MDM) requirements override local config.
