---
name: Trellis Create & Assign Maintenance Task
description: Create a maintenance or field task in Trellis against a property and department, assign it, then mark it complete with a summary.
api: Trellis Public API
base_url: https://app.trellistech.com/api/v1
auth: Bearer workspace API key (trls_...)
operations:
  - createTask   # POST  /workspaces/{workspaceId}/tasks
  - getTask      # GET   /workspaces/{workspaceId}/tasks/{taskId}
  - updateTask   # PATCH /workspaces/{workspaceId}/tasks/{taskId}
generated: '2026-07-21'
method: generated
source: https://docs.trellistech.com/api-reference/public-rest-api
---

# Trellis Create & Assign Maintenance Task

Open a task, assign it, and close it out. Writes run through the same mutation pipeline as
the Trellis app (assignees, tags, activity, notifications, automations, connected sync).

## Preconditions
- Workspace API key (`trls_...`) for the target workspace.
- A `departmentId` (UUID) that owns the task — `departmentId` is REQUIRED on create.
- Optionally a `propertyId` (UUID); or pass `propertyName` when the property id is unknown.

## Steps
1. Create the task with `createTask` (required: `title`, `departmentId`):
   ```bash
   curl -X POST -H "Authorization: Bearer $TRELLIS_API_KEY" -H "Content-Type: application/json" \
     -d '{"title":"Replace bathroom light bulb","departmentId":"33333333-3333-4333-8333-333333333333","priority":"HIGH","propertyId":"11111111-1111-4111-8111-111111111111"}' \
     "https://app.trellistech.com/api/v1/workspaces/{workspaceId}/tasks"
   ```
   Capture `id` and `shortId` (e.g. `TK-1234`) from the `201` response.
2. Assign on create with `primaryUserId` / `primaryVendorOrgId` / `secondaryAssigneeIds`,
   or set `useDefaultAssignees: true` to apply the department defaults.
3. Track status with `getTask`; valid statuses include `OPEN`, `SCHEDULED`,
   `IN_PROGRESS`, `COMPLETED`. Priorities: `WATCH`, `LOWEST`, `LOW`, `NORMAL`, `HIGH`, `URGENT`.
4. Complete it with `updateTask`:
   ```bash
   curl -X PATCH -H "Authorization: Bearer $TRELLIS_API_KEY" -H "Content-Type: application/json" \
     -d '{"status":"COMPLETED","summary":"Bulb replaced and tested."}' \
     "https://app.trellistech.com/api/v1/workspaces/{workspaceId}/tasks/{taskId}"
   ```

## Notes & rules
- Confirm intent with the user before creating or completing a task (write action).
- There is NO idempotency key: do not blindly retry a `POST` that may have succeeded —
  re-list or `getTask` to check before retrying (see conventions/).
- Handle `409` (completion blocked by another task/workflow rule) and `422` (valid JSON
  that cannot be applied to current state) from the custom `{error,message,details}`
  envelope (see errors/).
