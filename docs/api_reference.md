# API Reference

This document covers two distinct API systems within Guardian Pro:

1. **Guardian Model API** - For integrating custom AI models with Guardian Pro
2. **Guardian Pro PACS Integration API** - For connecting PACS systems to Guardian Pro

---

# Guardian Model API

**Version 1.0** | API specification for Guardian-compatible model containers.

## Overview

Your Docker container receives DICOM studies as TAR archives and returns JSON predictions. Expose port **8000** with the endpoints below.

---

## Input Format

| Field | Requirement |
|-------|-------------|
| **Format** | TAR archive containing raw DICOM files |
| **File types** | `.dcm`, `.dicom`, or extensionless with DICM header |
| **Max size** | 8GB |

Your container handles all processing internally (windowing, normalization, inference).

---

## Endpoints

### `GET /health-check`

Returns service status.

**Response** `200 OK`
```json
{
  "status": "healthy",
  "model_loaded": true,
  "message": "Service ready for predictions"
}
```

**Response** `500` (if unhealthy)
```json
{
  "status": "unhealthy",
  "model_loaded": false,
  "message": "Model failed to load"
}
```

---

### `POST /predict`

Processes a DICOM study and returns predictions.

**Request**
```
Content-Type: multipart/form-data

study: <tar_file>              # Required
conf_threshold: 0.2            # Optional, float 0.0-1.0
```

**Response** `200 OK`
```json
{
  "generated_report": "Clinical findings summary.",
  "predictions": [
    {
      "abnormality": "pulmonary_embolism",
      "bounding_box": [0.45, 0.32, 0.12, 0.08],
      "confidence": 0.95,
      "study_uid": "1.2.840.113619.2.55.3.123456",
      "series_uid": "1.2.840.113619.2.55.3.123456.1",
      "instance_uid": "1.2.840.113619.2.55.3.123456.1.1",
      "icd10_code": "I26.9",
      "icd10_description": "Pulmonary embolism without acute cor pulmonale",
      "icd10_snippet": "PE detected. Recommend anticoagulation therapy."
    }
  ]
}
```

#### Prediction Fields

| Field | Type | Description |
|-------|------|-------------|
| `abnormality` | string | Condition name |
| `bounding_box` | array[4] | YOLO format `[x, y, w, h]`, normalized 0-1 |
| `confidence` | float | 0-1 |
| `study_uid` | string | DICOM StudyInstanceUID |
| `series_uid` | string | DICOM SeriesInstanceUID |
| `instance_uid` | string | DICOM SOPInstanceUID |
| `icd10_code` | string | ICD-10 diagnosis code |
| `icd10_description` | string | ICD-10 description |
| `icd10_snippet` | string | Clinical context snippet |

All fields are required.

---

### `GET /icd10-mapping`

Returns current ICD-10 mappings.

**Response** `200 OK`
```json
{
  "mappings": {
    "pulmonary_embolism": {
      "code": "I26.9",
      "description": "Pulmonary embolism without acute cor pulmonale",
      "icd10_snippet": "PE detected. Recommend anticoagulation therapy."
    }
  },
  "count": 1,
  "last_updated": "startup"
}
```

---

### `POST /icd10-mapping`

Updates ICD-10 mappings.

**Request**
```json
{
  "pulmonary_embolism": {
    "code": "I26.9",
    "description": "Pulmonary embolism without acute cor pulmonale",
    "icd10_snippet": "PE detected. Recommend anticoagulation therapy."
  }
}
```

**Response** `200 OK`
```json
{
  "status": "updated",
  "mappings_count": 1,
  "message": "ICD-10 mappings updated successfully"
}
```

---

### `GET /logging`

Returns container logs.

**Response** `200 OK`
```json
{
  "logs": [
    "2024-01-15 10:00:00 INFO: Model loaded successfully",
    "2024-01-15 10:01:23 INFO: Processing study..."
  ],
  "log_count": 2,
  "timestamp": "2024-01-15T10:01:30Z"
}
```

---

## Error Responses

**`400 Bad Request`**
```json
{
  "detail": "Invalid input: TAR file contains no valid DICOM files"
}
```

**`500 Internal Server Error`**
```json
{
  "detail": "Internal server error: Model inference failed"
}
```

---

## Performance Requirements

| Metric | Requirement |
|--------|-------------|
| Inference time | ≤ 30 seconds per study |
| Memory | ≤ 8GB RAM |
| Health check | < 5 seconds |

---

# Guardian Pro PACS Integration API

## Overview

Guardian Pro integrates directly with your PACS system, allowing radiologists to flag cases for peer review in real-time through custom PACS buttons or workflows. When clicked, the button opens a web browser and navigates to the Guardian Pro interface.

## Button Manual Trigger

### Flag Case API Endpoints

#### Deep Link — Create

This endpoint creates a flagged case and redirects the user to the Guardian Pro interface.

**Endpoint**

Behind Traefik (TLS): `https://<GUARDIAN_DOMAIN>`

```
GET /flag-case?accession={accession}&user_name={user_name}
```

**Parameters**

| Parameter | Type | Description | Required |
|-----------|------|-------------|----------|
| `accession` | string | Accession number of the study to flag | Yes |
| `user_name` | string | Username or identifier of the radiologist flagging the case | Yes |

**Example**

```
GET https://guardian.yourhospital.com/flag-case?accession=ACC123456&user_name=drsmith
```

**Response**

The endpoint opens in a web browser and redirects the user to the Guardian Pro viewer where they can:
- Review the flagged study
- Add comments or notes
- Submit the case for peer review

**PACS Button Configuration**

Configure your PACS to add a custom button with the following behavior:
1. Extract the current study's accession number
2. Extract the current user's username
3. Open URL in default web browser: `https://<GUARDIAN_DOMAIN>/flag-case?accession={accession}&user_name={user_name}`

**User Experience**

When a radiologist clicks the button in their PACS application:
1. Their default web browser opens (or a new tab if browser is already open)
2. Browser navigates to the Guardian Pro interface with the study pre-loaded
3. Radiologist can flag the case and add relevant notes
4. Radiologist can return to their PACS workflow

---

## Guardian Pro PACS Worklist Integration (Bidirectional Event Sync)

### Overview

Guardian Pro supports bidirectional PACS worklist synchronization using event-driven APIs.

This integration allows PACS and Guardian Pro to exchange create/update/delete events for:

- `peer_review` tasks
- `self_review` tasks
- `interesting_case` tasks

Use this integration when both systems must stay in sync without duplicate manual task completion.

---

### Authentication

All PACS worklist endpoints require a shared API key header:

- Header: `X-Guardian-Api-Key`
- Value: configured in Guardian Pro (`GUARDIAN_PACS_SYNC_API_KEY`)

Requests without a valid key return `401 Unauthorized`.

---

### Event Types and Actions

#### Event Types
- `peer_review`
- `self_review`
- `interesting_case`

#### Actions
- `created`
- `updated`
- `deleted`

---

### Inbound Endpoints (PACS -> Guardian)

Base path:

`/api/v1/pacs`

#### Create/Update/Delete Peer Review
`POST /api/v1/pacs/events/peer-review`

#### Create/Update/Delete Self Review
`POST /api/v1/pacs/events/self-review`

#### Create/Update/Delete Interesting Case
`POST /api/v1/pacs/events/interesting-case`

#### Queue Health
`GET /api/v1/pacs/events/health`

#### Manual Dispatch Trigger (optional operational endpoint)
`POST /api/v1/pacs/events/dispatch`

---

### Outbound Delivery (Guardian -> PACS)

Guardian Pro emits outbound events to PACS via an outbox dispatcher to:

- `GUARDIAN_PACS_SYNC_OUTBOUND_URL`

Outbound events use the same canonical event envelope (`event_type`, `action`, `study_uid`, etc.) and are retried automatically on transient failure.

---

### Event Payload Contract

#### Request Body (Inbound)
```json
{
  "source_system": "agfa_pacs",
  "event_id": "evt-123456",
  "occurred_at": "2026-03-09T12:34:56Z",
  "event_type": "peer_review",
  "action": "created",
  "study_uid": "1.2.840.113619.2.55.3.123456",
  "payload": {
    "review_event_id": 12345,
    "determination": 0,
    "comments": "Optional comments"
  }
}
```

#### Response
```json
{
  "accepted": true,
  "duplicate": false,
  "message": "Event processed",
  "event_id": "evt-123456",
  "study_uid": "1.2.840.113619.2.55.3.123456",
  "event_type": "peer_review",
  "action": "created"
}
```

---

### Idempotency and Retries

- Idempotency key: `(source_system, event_id)`
- Duplicate inbound events are acknowledged with `duplicate: true`
- Outbound delivery uses retry/backoff with queued status tracking (`pending`, `retrying`, `failed`, `sent`)

---

### Configuration

Set these in Guardian Pro environment:

- `GUARDIAN_PACS_SYNC_ENABLED=true`
- `GUARDIAN_PACS_SYNC_API_KEY=<shared-secret>`
- `GUARDIAN_PACS_SYNC_OUTBOUND_URL=https://<pacs-endpoint>/...`
- `GUARDIAN_PACS_SYNC_DISPATCH_INTERVAL_SECONDS`
- `GUARDIAN_PACS_SYNC_DISPATCH_BATCH_SIZE`
- `GUARDIAN_PACS_SYNC_REQUEST_TIMEOUT_SECONDS`
- `GUARDIAN_PACS_SYNC_MAX_ATTEMPTS`
- `GUARDIAN_PACS_SYNC_RETRY_BASE_SECONDS`
- `GUARDIAN_PACS_SYNC_RETRY_MAX_SECONDS`
- `GUARDIAN_PACS_SYNC_INTERESTING_POLL_SECONDS`

---

### Integration Steps

1. Enable PACS sync config and deploy Guardian Pro.
2. Configure PACS to call inbound Guardian endpoints for `peer_review`, `self_review`, and `interesting_case`.
3. Configure Guardian outbound URL to PACS receiver.
4. Use unique `event_id` values per source system for idempotency.
5. Validate with test events for all actions (`created`, `updated`, `deleted`) and event types.
6. Monitor `/api/v1/pacs/events/health` during go-live.

---

### Testing (curl examples)

#### Inbound peer review example
```bash
curl -X POST "https://<GUARDIAN_DOMAIN>/api/v1/pacs/events/peer-review" \
  -H "Content-Type: application/json" \
  -H "X-Guardian-Api-Key: <shared-secret>" \
  -d '{
    "source_system":"agfa_pacs",
    "event_id":"evt-pr-001",
    "occurred_at":"2026-03-09T15:00:00Z",
    "event_type":"peer_review",
    "action":"created",
    "study_uid":"1.2.840.113619.2.55.3.123456",
    "payload":{"determination":0}
  }'
```

#### Inbound self review example
```bash
curl -X POST "https://<GUARDIAN_DOMAIN>/api/v1/pacs/events/self-review" \
  -H "Content-Type: application/json" \
  -H "X-Guardian-Api-Key: <shared-secret>" \
  -d '{
    "source_system":"agfa_pacs",
    "event_id":"evt-sr-001",
    "occurred_at":"2026-03-09T15:01:00Z",
    "event_type":"self_review",
    "action":"updated",
    "study_uid":"1.2.840.113619.2.55.3.123456",
    "payload":{"determination":1}
  }'
```

#### Inbound interesting case example
```bash
curl -X POST "https://<GUARDIAN_DOMAIN>/api/v1/pacs/events/interesting-case" \
  -H "Content-Type: application/json" \
  -H "X-Guardian-Api-Key: <shared-secret>" \
  -d '{
    "source_system":"agfa_pacs",
    "event_id":"evt-ic-001",
    "occurred_at":"2026-03-09T15:02:00Z",
    "event_type":"interesting_case",
    "action":"deleted",
    "study_uid":"1.2.840.113619.2.55.3.123456",
    "payload":{}
  }'
```

#### Dispatch health
```bash
curl -H "X-Guardian-Api-Key: <shared-secret>" \
  "https://<GUARDIAN_DOMAIN>/api/v1/pacs/events/health"
```

---

### Vendor Implementation Checklist (AGFA/PACS)

Use this checklist during implementation, validation, and go-live.

#### 1) Connectivity and Security
- [ ] Guardian endpoint is reachable from PACS network: `https://<GUARDIAN_DOMAIN>/api/v1/pacs/...`
- [ ] PACS outbound TLS trust chain is validated for Guardian certificate
- [ ] Shared API key is configured on both sides (`X-Guardian-Api-Key`)
- [ ] API key rotation and secret ownership are documented

#### 2) Event Contract Mapping
- [ ] PACS emits `event_type` values only from: `peer_review`, `self_review`, `interesting_case`
- [ ] PACS emits `action` values only from: `created`, `updated`, `deleted`
- [ ] `study_uid` is populated with DICOM `StudyInstanceUID`
- [ ] `occurred_at` is sent in UTC ISO-8601 format (for example: `2026-03-09T15:00:00Z`)
- [ ] `source_system` is stable and environment-specific (for example: `agfa_pacs_prod`)

#### 3) Idempotency and Ordering
- [ ] `event_id` is globally unique per `source_system`
- [ ] Retry from PACS preserves the same `event_id` (do not generate a new ID on retry)
- [ ] Duplicate events are treated as success when Guardian responds with `duplicate: true`
- [ ] Out-of-order updates/deletes are handled according to PACS business rules

#### 4) Inbound Endpoint Coverage
- [ ] `POST /api/v1/pacs/events/peer-review` tested for `created`, `updated`, `deleted`
- [ ] `POST /api/v1/pacs/events/self-review` tested for `created`, `updated`, `deleted`
- [ ] `POST /api/v1/pacs/events/interesting-case` tested for `created`, `updated`, `deleted`
- [ ] Negative tests validated (`401`, malformed JSON, missing required fields)

#### 5) Outbound Receiver Readiness (Guardian -> PACS)
- [ ] PACS receiver URL is deployed and reachable at `GUARDIAN_PACS_SYNC_OUTBOUND_URL`
- [ ] PACS receiver accepts the canonical Guardian event envelope
- [ ] PACS receiver returns clear HTTP status codes (`2xx` success, `4xx/5xx` failure)
- [ ] PACS receiver is idempotent for repeated delivery attempts

#### 6) Operational Controls
- [ ] `GET /api/v1/pacs/events/health` is included in PACS/ops monitoring
- [ ] Manual dispatch endpoint is secured and documented: `POST /api/v1/pacs/events/dispatch`
- [ ] Guardian retry and backoff settings are tuned for site SLAs
- [ ] Alerting is configured for sustained queue failures (`failed`, high retry volume)

#### 7) Go-Live Validation
- [ ] End-to-end test completed in non-production with all event types and actions
- [ ] Production dry run completed with a limited pilot group
- [ ] Rollback procedure and contact matrix are documented
- [ ] Post go-live monitoring window and ownership are confirmed

---

## Guardian Model API Testing

```bash
# Health check
curl http://localhost:8000/health-check

# Prediction
curl -X POST http://localhost:8000/predict \
  -F "study=@study.tar" \
  -F "conf_threshold=0.3"

# Get mappings
curl http://localhost:8000/icd10-mapping

# Logs
curl http://localhost:8000/logging
```

---

## Guardian Pro PACS Integration Testing

```bash
# Test flag case endpoint (replace with your domain and parameters)
curl "https://guardian.yourhospital.com/flag-case?accession=ACC123456&user_name=drsmith"
```
