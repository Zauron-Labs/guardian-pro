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
