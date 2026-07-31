---
name: Check a vehicle's connectivity before requesting data
description: Verify that a VIN is connectable via Otonomo before attempting to read its status, then pull per-attribute status.
api: openapi/otonomo-fleet-openapi.yml
operations: [workspace-access-token-eu, connectivity-check, vehicle-status-time-per-attribute]
---

# Check a vehicle's connectivity before requesting data

Before onboarding or polling a vehicle, verify Otonomo can actually reach it — this avoids wasted status calls on unsupported/unconnected VINs.

## 1. Authenticate
`POST /v1/oauth/token/` (`workspace-access-token-eu`). Form body: `client_id`, `client_secret`, `grant_type=client_credentials`, `service`. Send `Authorization: Bearer <access_token>` thereafter.

## 2. Check connectivity
`GET /vehicle-connectivity/{vin}` (operation `connectivity-check`; US variant `connectivity-check-us`). Confirms whether the vehicle's make/model/year is connectable and what capabilities are available.

## 3. Read per-attribute status
If connectable, `GET /v2/vehicles/{vin}/status` (operation `vehicle-status-time-per-attribute`) returns each attribute with the exact timestamp Otonomo received it (unlike `vehicle-status-for-fleets` v1, which aggregates). Attribute keys follow the `domain__group__field` convention.

## Rules
- Only request status for VINs that pass the connectivity check.
- A `403 {"detail": "..."}` indicates the token scope/region does not cover the VIN.
- Honor rate limits; back off on `429`.
