# Test Contracts

## Contract Tests
- **Authentication**: Verify `/auth/v1/token` exchange returns session tokens usable for Supabase requests and AI token minting.
- **Session Upload**: Validate schema for `POST /v1/sessions` and ensure duplicate session keys return 409.
- **Sample Batch**: Confirm sample payload schema, checksum validation, and partial failure response handling for `POST /v1/sessions/{session_id}/samples`.
- **Metrics Update**: Ensure `PUT /v1/sessions/{session_id}/metrics` enforces required fields and rejects inconsistent durations.
- **AI Request Payload**: Validate JSON body sent to AI endpoint includes metrics summary, trend deltas, and anonymized identifiers only.

## Integration Tests
- **Measurement Happy Path**: Launch end-to-end flow from login to upload with mocked BLE device verifying <1s latency budgets.
- **Offline Retry**: Simulate connectivity loss after measurement; confirm queue persists payloads and retries successfully.
- **History Trend Aggregation**: Seed database with multi-week sessions; verify weekly/monthly filters and sparklines render expected statistics.
- **Caregiver Account Access**: Test delegated access to patient sessions with correct RBAC enforcement.
- **AI Consultation**: Validate prompt packaging and display of streaming responses in UI.

## Performance Benchmarks
- **Chart Rendering**: Ensure 20-second measurement renders within 200 ms on target devices.
- **Metric Computation**: Benchmark metrics calculator to complete within 1 second using 50 Hz sample set.
- **Upload Pipeline**: Measure round-trip latency from session completion to server acknowledgement under 1 second on median network.
