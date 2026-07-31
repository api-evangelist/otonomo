---
name: Generate and download an asynchronous fleet trips report
description: Kick off an asynchronous Otonomo fleet trips report, poll for completion, and download the resulting CSV.
api: openapi/otonomo-fleet-openapi.yml
operations: [workspace-access-token-eu, personal-trips-data-historical, get-report-status-eu]
---

# Generate and download an asynchronous fleet trips report

Otonomo delivers historical/bulk data asynchronously: you request a report, receive a request id, poll until it is ready, then download a CSV.

## 1. Authenticate
`POST /v1/oauth/token/` (`workspace-access-token-eu`). Form body: `client_id`, `client_secret`, `grant_type=client_credentials`, `service`. Use `Authorization: Bearer <access_token>` thereafter.

## 2. Request the trips report
`POST /v1/personal/reports/trips` (operation `personal-trips-data-historical`). Provide the vehicle/geography/time window. The response returns a **request id**. (Points reports use `fleet-points-historical-data` at `/v1/personal/reports/points`; trips points use `fleet-trips-points`; bulk status uses `bulk-vehicles-status-for-fleets-eu`.) Note: trips/points reports are a **premium** feature — contact support@otonomo.io if you get a permission error.

## 3. Poll for completion
`GET /v1/personal/reports/requests/{Request_id}` (operation `get-report-status-eu`) with the request id. Status is one of:
- `Ongoing` — keep polling (with backoff);
- `Completed` — a link to the generated CSV is included in the response;
- `Expired` — the report was generated over 30 days ago; regenerate it.

## 4. Download
When status is `Completed`, fetch the CSV from the link in the response.

## Rules
- Reports expire 30 days after generation — download promptly.
- Poll with exponential backoff; do not tight-loop the status endpoint (respect rate limits, back off on `429`).
- Each report call creates a new request id (no idempotency) — avoid duplicate submissions by tracking the returned id.
