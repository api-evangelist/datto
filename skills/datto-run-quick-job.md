---
name: Datto RMM run a quick job
description: Schedule a quick job (component) on a device and read back its results and stdout/stderr.
api: openapi/datto-rmm-openapi.json
operations: [getComponents, createQuickJob, get_1, getJobResults, getStdOut, getStdErr]
---

# Datto RMM: run a quick job on a device

Dispatch an on-demand component ("quick job") to a device and collect its output.

## Preconditions
- OAuth 2.0 bearer token (see authentication/datto-authentication.yml).
- The target `deviceUid` (from the device-inventory skill).

## Steps
1. **Pick a component** — `getComponents` (`GET /v2/account/components`) to find the
   componentUid you want to run.
2. **Schedule the job** — `createQuickJob`
   (`PUT /v2/device/{deviceUid}/quickjob`) with the job name and component. The
   response returns the created job's identifier.
3. **Poll the job** — `get_1` (`GET /v2/job/{jobUid}`) until the job completes.
4. **Read results** — `getJobResults`
   (`GET /v2/job/{jobUid}/results/{deviceUid}`); then `getStdOut`
   (`.../stdout`) and `getStdErr` (`.../stderr`) for component output.

## Rules
- `createQuickJob` is a write operation (100 / rolling 60s limit) and is NOT
  idempotency-key protected — if the PUT times out, poll for an existing job before
  re-issuing to avoid double-dispatch (see conventions/datto-conventions.yml).
- Poll `get_1` with backoff; do not tight-loop against the 600/60s read limit.
- A 400 on `createQuickJob` means invalid job data; 403 means insufficient rights
  on the device's site.
