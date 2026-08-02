---
name: Trellis Daily Work-Orders Report
description: Pull a costed daily work-order report for a Trellis workspace and calendar date, summarizing scheduled field-ops tasks with assignees and cost totals.
api: Trellis Public API
base_url: https://app.trellistech.com/api/v1
auth: Bearer workspace API key (trls_...)
operations:
  - listDailyWorkOrders   # GET /workspaces/{workspaceId}/tasks/daily-work-orders
generated: '2026-07-21'
method: generated
source: https://docs.trellistech.com/api-reference/public-rest-api
---

# Trellis Daily Work-Orders Report

Produce the day's field-operations work orders for a workspace, with per-task
assignees, cost items, and per-currency totals.

## Preconditions
- A workspace API key (`trls_...`) issued for the target workspace (Settings > Developer).
- The `{workspaceId}` in the path MUST match the key's workspace, or the call returns `401`.

## Steps
1. Choose the local calendar date to export, formatted `YYYY-MM-DD`.
2. Call `listDailyWorkOrders`:
   ```bash
   curl -H "Authorization: Bearer $TRELLIS_API_KEY" \
     "https://app.trellistech.com/api/v1/workspaces/{workspaceId}/tasks/daily-work-orders?date=2026-06-18"
   ```
3. Read the response: `workOrderCount`, `totalCost`, `totalCostCurrency`, and
   `costTotalsByCurrency`, then iterate `workOrders[]` for per-task
   `property`, `department`, `scheduledTime`, `costItems[]`, and `assignees[]`.
4. If `totalCost` is `null` and `totalCostCurrency` is `"MIXED"`, report costs from
   `costTotalsByCurrency` per currency (do not sum across currencies).

## Notes & rules
- Only `SCHEDULED` tasks in `field_ops` departments for that date are included; other
  statuses/departments are excluded by design.
- This endpoint is read-only. No idempotency key is needed (see conventions/).
- Error envelope is custom JSON `{error, message, details}` (see errors/); handle `401`
  (key/workspace mismatch), `404` (workspace not found), and `500` (retry).
