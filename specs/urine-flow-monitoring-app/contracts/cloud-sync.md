# Cloud Sync Contract

## Authentication
- Mobile client authenticates via Supabase Auth, receiving a short-lived access token.
- Backend issues temporary device tokens (JWT) when initiating measurements to authorize sensor communication.

## Session Upload Endpoint
- **URL**: `POST /v1/sessions`
- **Request Body**:
```
{
  "session_key": "string",
  "user_id": "uuid",
  "device_id": "uuid",
  "start_ts": "2025-09-17T08:30:00Z",
  "duration_s": 20,
  "status": "completed|aborted|failed",
  "notes": "optional"
}
```
- **Response**: `201 Created` with `{ "session_id": "uuid" }`

## Sample Batch Upload
- **URL**: `POST /v1/sessions/{session_id}/samples`
- **Request Body**:
```
{
  "samples": [
    { "t_ms": 0, "flow_ml_s": 0.0, "quality": "good" },
    { "t_ms": 50, "flow_ml_s": 2.4, "quality": "good" }
  ],
  "chunk_index": 0,
  "chunk_checksum": "sha256"
}
```
- **Response**: `202 Accepted` with acknowledgement token used to mark local queue items as synced.

## Metrics Upload
- **URL**: `PUT /v1/sessions/{session_id}/metrics`
- **Request Body**:
```
{
  "q_max": 12.3,
  "q_avg": 7.8,
  "voided_volume_est": 320,
  "time_to_peak": 3500,
  "flow_time_s": 18.2,
  "duration_s": 20.0,
  "tq_curve_hash": "abcdef",
  "quality_flags": ["LOSS_UNDER_THRESHOLD"]
}
```
- **Response**: `200 OK`

## Acknowledgement Endpoint
- **URL**: `POST /v1/sessions/{session_id}/ack`
- **Request Body**: `{ "ack_token": "string", "status": "synced|rejected", "reason": "optional" }`
- **Purpose**: Confirms receipt of samples and metrics; rejected payloads must supply reason for retry logging.

## Error Codes
- `409 CONFLICT`: Duplicate session key or mismatch between mobile and backend metadata.
- `422 UNPROCESSABLE_ENTITY`: Validation failures (e.g., samples outside expected range).
- `500 INTERNAL_SERVER_ERROR`: Unexpected backend failure; client should retry with exponential backoff.

## Security Requirements
- All endpoints served over TLS.
- Access tokens validated against RBAC roles (patient, caregiver, support).
- Audit logs must capture session creation, metric upload, and deletion requests with user identifiers.
