# Device Protocol Contract

## BLE Service Definition
- **Service UUID**: `urineflow-svc`
- **Characteristics**:
  - `cmd` (write): accepts ASCII commands initiated by the mobile app.
  - `stream` (notify): emits JSON payloads with telemetry samples.

## Start Command
```
START{session_key:<string>,user_id_hash:<string>,duration:20,ts_ms:<epoch_ms>}
```
- `session_key`: generated per measurement session and shared with backend for validation.
- `user_id_hash`: hashed identifier to avoid exposing PII to the device.
- `duration`: fixed at 20 seconds for v1.
- `ts_ms`: device-app synchronized epoch timestamp in milliseconds.

## Stream Payload
```
{
  "t_ms": <int>,            // elapsed milliseconds since session start
  "flow_ml_s": <double>,    // instantaneous flow rate
  "quality": <string>       // good | interpolated | dropped | warning
}
```
- Frequency: 20-50 Hz.
- Device must continue sending until `duration` elapses or an abort command is received.
- Quality flag semantics must align with metrics calculator expectations (see `metrics_calculator_test.dart`).

## Abort Command
```
ABORT{session_key:<string>,reason:<string>}
```
- Used when the user cancels measurement or error thresholds exceeded.

## Error Handling
- Device should acknowledge invalid commands with `ERR{code,message}` payloads on the `stream` characteristic.
- App must treat missing acknowledgements as retry triggers and surface errors to the user.
- Time synchronization should occur at pairing; if clock skew >250 ms, device returns `ERR{code:"CLOCK_SKEW"}`.

## MQTT/WebSocket Variant (Future)
- **Command Topic**: `v1/{device_id}/cmd`
- **Stream Topic**: `v1/{device_id}/stream/{session_key}`
- **Payload**: JSON equivalent to BLE protocol with JWT/HMAC signature appended as `token`.
