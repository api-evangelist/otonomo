---
name: Onboard fleet vehicles and read their status
description: Add VINs to an Otonomo Workspace, request driver/vehicle enablement, confirm they are active, then read near-real-time vehicle status.
api: openapi/otonomo-fleet-openapi.yml
operations: [workspace-access-token-eu, vehicle-management-upload-new-vin, vehicle-management-request-vins-consent-1, vehicle-management-get-all-vins, vehicle-status-for-fleets]
---

# Onboard fleet vehicles and read their status

Use the Otonomo Fleet API to onboard vehicles into a Workspace and pull their latest status. Pick the correct region host: **US `https://api.otonomo.io`**, **EU `https://api.eu.otonomo.io`**.

## 1. Authenticate
`POST /v1/oauth/token/` (operation `workspace-access-token-eu` for EU / `service-access-token` for US). Send a form body: `client_id`, `client_secret`, `grant_type=client_credentials`, and your `service` (Service ID). You get back `{ access_token, expires_in: 86400, token_type: "Bearer" }`. Send `Authorization: Bearer <access_token>` on every subsequent call.

## 2. Upload VINs
`POST /v2/vehicle-management/vehicles/` (operation `vehicle-management-upload-new-vin`). Provide the VINs plus optional static data (labels, name, license_plate, color, model, year, fuel_type, fuel_capacity, battery_capacity).

## 3. Request enablement / consent
`POST /v2/vehicle-management/vehicles/enable` (operation `vehicle-management-request-vins-consent-1`) to request enablement for disabled VINs. For private-ownership vehicles, driver consent must be obtained first (see docs: obtaining-driver-consent).

## 4. Confirm status of the fleet
`GET /v2/vehicle-management/vehicles/` (operation `vehicle-management-get-all-vins`) returns the current onboarding status of every VIN in the Workspace. Only act on VINs that are active/enabled.

## 5. Read a vehicle's live status
`GET /v1/vehicles/{vin}/status` (operation `vehicle-status-for-fleets`). Response fields use the namespaced double-underscore convention, e.g. `vehicle__battery__percentage`, `location__latitude__value`, `mobility__speed__value`, `metadata__time__epoch`. Use `vehicle-status-time-per-attribute` (`GET /v2/vehicles/{vin}/status`) when you need a per-attribute timestamp instead of an aggregated one.

## Rules
- Tokens expire after 86400s — cache and refresh; do not request a new token per call.
- A `403 {"detail": "..."}` means the token's Workspace/service does not cover that VIN or region — re-check the Service ID and region host, do not retry blindly.
- Respect the documented API rate limits (https://docs.otonomo.io/docs/api-rate-limits); back off on `429`.
- There is no idempotency-key mechanism; vehicle-management writes are not safe to blindly retry.
