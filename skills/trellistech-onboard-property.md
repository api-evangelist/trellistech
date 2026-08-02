---
name: Trellis Onboard Property
description: Create a new short-term rental property in Trellis and fill in its operating details (check-in times, WiFi, instructions, fees), then verify it in the workspace listing.
api: Trellis Public API
base_url: https://app.trellistech.com/api/v1
auth: Bearer workspace API key (trls_...)
operations:
  - createProperty   # POST  /workspaces/{workspaceId}/properties
  - updateProperty   # PATCH /workspaces/{workspaceId}/properties/{propertyId}
  - getProperty      # GET   /workspaces/{workspaceId}/properties/{propertyId}
  - listProperties   # GET   /workspaces/{workspaceId}/properties
generated: '2026-07-21'
method: generated
source: https://docs.trellistech.com/api-reference/public-rest-api
---

# Trellis Onboard Property

Register a property and complete the operating details operators rely on for guest stays.

## Preconditions
- Workspace API key (`trls_...`) for the target workspace.

## Steps
1. Create the property with `createProperty` (required: `name`; useful: `status`, `city`):
   ```bash
   curl -X POST -H "Authorization: Bearer $TRELLIS_API_KEY" -H "Content-Type: application/json" \
     -d '{"name":"Casa Duomo","status":"ONBOARDING","city":"Milano"}' \
     "https://app.trellistech.com/api/v1/workspaces/{workspaceId}/properties"
   ```
   Capture `id` from the `201` response. Valid `status` values: `PROSPECT`, `ONBOARDING`,
   `ACTIVE`, `AT_RISK`, `INACTIVE`.
2. Fill in operating detail with `updateProperty` (check-in/out, WiFi, instructions, fees):
   ```bash
   curl -X PATCH -H "Authorization: Bearer $TRELLIS_API_KEY" -H "Content-Type: application/json" \
     -d '{"wifiName":"Casa Duomo Guest","checkinTime":"15:00","checkoutTime":"10:00","timezone":"Europe/Rome"}' \
     "https://app.trellistech.com/api/v1/workspaces/{workspaceId}/properties/{propertyId}"
   ```
3. When onboarding is complete, set `status` to `ACTIVE` via `updateProperty`.
4. Verify with `getProperty`, or find it in `listProperties?status=ACTIVE&q=duomo`
   (search matches name, internal code, or city).

## Notes & rules
- Confirm intent before create/update (write actions); no idempotency key exists (conventions/).
- Store workspace-specific attributes in `customFields`.
- List/get responses paginate with `{items, pagination:{total,limit,offset,hasMore}}`
  (`limit` default 50, max 100).
- Errors use the custom `{error,message,details}` envelope; handle `400`/`422` validation
  and `404` (workspace/property not found) — see errors/.
