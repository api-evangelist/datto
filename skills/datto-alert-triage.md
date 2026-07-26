---
name: Datto RMM alert triage
description: Pull open monitoring alerts across the account or a site, inspect one, then mute or resolve it.
api: openapi/datto-rmm-openapi.json
operations: [getUserAccountOpenAlerts, getSiteOpenAlerts, getAlert, muteAlert, resolveAlert, unmuteAlert]
---

# Datto RMM: alert triage

Triage RMM monitoring alerts and take action.

## Preconditions
- OAuth 2.0 bearer token (see authentication/datto-authentication.yml). Use your
  account's regional CentraStage host.

## Steps
1. **List open alerts** — account-wide via `getUserAccountOpenAlerts`
   (`GET /v2/account/alerts/open`) or per site via `getSiteOpenAlerts`
   (`GET /v2/site/{siteUid}/alerts/open`). Page with `page`/`max` (max 250).
2. **Inspect an alert** — `getAlert` (`GET /v2/alert/{alertUid}`) for the priority,
   monitor type, and originating device.
3. **Act**:
   - `muteAlert` (`POST /v2/alert/{alertUid}/mute`) to suppress noise; reverse with
     `unmuteAlert` (`POST /v2/alert/{alertUid}/unmute`).
   - `resolveAlert` (`POST /v2/alert/{alertUid}/resolve`) once handled.

## Rules
- Mute/resolve/unmute are write operations — the write limit is 100 / rolling 60s.
- These POSTs are not idempotent-key protected; do not blind-retry on timeout,
  re-read the alert state first (see conventions/datto-conventions.yml).
- 401 means the token is missing/expired; 403 means the RMM user lacks permission
  on that alert's site.
