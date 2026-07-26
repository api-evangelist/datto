---
name: Datto RMM device inventory
description: Enumerate an MSP account's sites and devices and pull hardware/software audit detail for a device.
api: openapi/datto-rmm-openapi.json
operations: [getSites, getSiteDevices, getByUid, getDeviceAudit, getDeviceAuditSoftware]
---

# Datto RMM: device inventory

Use the Datto RMM v2 REST API to build a current inventory of managed devices.

## Preconditions
- Obtain an OAuth 2.0 bearer token by exchanging your RMM API key + secret at
  `https://[platform]-api.centrastage.net/auth/oauth/token` (token TTL 100 hours).
- Send it as `Authorization: Bearer <token>`. Use your account's regional host
  (concord/merlot/pinotage/vidal/zinfandel/syrah).

## Steps
1. **List sites** — `getSites` (`GET /v2/account/sites`). Page with `page`/`max`
   (max 250 per page); follow `pageDetails` to page through all sites.
2. **List devices per site** — `getSiteDevices` (`GET /v2/site/{siteUid}/devices`).
   Same pagination. Optionally filter with `hostname`, `deviceType`, or `filterId`.
3. **Fetch a device** — `getByUid` (`GET /v2/device/{deviceUid}`) for status,
   last-seen, and identifiers.
4. **Audit detail** — `getDeviceAudit` (`GET /v2/audit/device/{deviceUid}`) for
   hardware/OS; `getDeviceAuditSoftware` (`GET /v2/audit/device/{deviceUid}/software`)
   for installed software.

## Rules
- Respect the read rate limit: 600 requests / rolling 60s. Check
  `GET /v2/system/request_rate` if you approach it; back off on HTTP 429.
- Timestamps are Unix epoch milliseconds.
- A 400 on an audit call means the device is not of the expected class (e.g. asking
  for a printer audit on a non-printer) — see errors/datto-problem-types.yml.
