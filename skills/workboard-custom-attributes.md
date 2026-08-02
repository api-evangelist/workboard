---
name: Manage custom attributes in WorkBoard
description: Discover an org's custom field definitions and read/set attribute values on objectives, key results, work items, and users via the Public API v2.
api: openapi/public-v2-openapi-original.yml
operations:
  - CustomAttributesController_getDefinitions
  - CustomAttributesController_getObjectsForAttribute
  - CustomAttributesController_getObjectiveAttributes
  - CustomAttributesController_createObjectiveAttributeValue
  - CustomAttributesController_setObjectiveAttributeValue
  - CustomAttributesController_deleteObjectiveAttributeValue
---

# Manage custom attributes in WorkBoard

## Auth

Send `Authorization: Bearer <token>` (instant token or OAuth access token).
API root: `https://www.myworkboard.com/wb/apis/v2`.

## Steps

1. **List the org's definitions.** `CustomAttributesController_getDefinitions`
   (`GET /attributes/definitions`) returns every custom field definition —
   note the `attributeId` you need.
2. **Find objects carrying a value.**
   `CustomAttributesController_getObjectsForAttribute`
   (`GET /attributes/{attributeId}/objects`) lists every object with a value
   for that attribute. Paginate with `limit`/`offset`; the response's
   `pagination.nextOffset` is the offset for the next page and is `null` on
   the final page.
3. **Read values on one object.**
   `CustomAttributesController_getObjectiveAttributes`
   (`GET /attributes/objectives/{id}`) — sibling endpoints exist for
   `key-results`, `work-items`, and `users`.
4. **Create vs update.** POST
   (`CustomAttributesController_createObjectiveAttributeValue`) sets a value
   that does not exist yet; PUT
   (`CustomAttributesController_setObjectiveAttributeValue`) updates an
   existing one; DELETE
   (`CustomAttributesController_deleteObjectiveAttributeValue`) clears it
   (`204` on success).

## Rules

- Errors come back in the `ApiErrorResponse` envelope
  (`statusCode`/`error`/`message`/`code`/`details`); branch on the stable
  machine code (e.g. `WB_REQUEST_VALIDATION_FAILED`) and inspect
  `details.invalidValues` for rejected attribute values
  (`errors/workboard-problem-types.yml`).
- `403` means the caller lacks permission on the target object; `404` means
  the definition or object does not exist.
- No idempotency keys — a retried POST may conflict; prefer PUT for
  replays (`conventions/workboard-conventions.yml`).
