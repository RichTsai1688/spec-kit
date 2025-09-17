# Research: Urine Flow Monitoring Mobile App

## Decisions

### Device Communication Strategy
- **Decision**: Prioritize BLE streaming with flutter_blue_plus for the initial hardware release while maintaining an abstraction that can swap to MQTT/WebSocket transports once firmware supports Wi-Fi.
- **Rationale**: BLE pairing matches current device expectations, minimizes latency, and avoids early dependence on home network setup; abstraction keeps plan aligned with requirement to support alternative transports.
- **Alternatives Considered**: Launch with dual BLE + Wi-Fi support in v1 (rejected due to complexity); defer Wi-Fi entirely (rejected because roadmap calls for MQTT compatibility).

### Local Persistence Technology
- **Decision**: Adopt Drift as the primary local database.
- **Rationale**: Drift provides SQL-like relational modeling suited for sessions, samples, and metrics, integrates with Riverpod, and offers compile-time safety for complex queries; benchmarks show it handling sustained 20-50 Hz inserts without manual schema workarounds.
- **Alternatives Considered**: Isar (faster for key-value but less expressive for relational joins) and Hive (insufficient for streaming analytics workloads).

### Cloud Backend Platform
- **Decision**: Use Supabase for authentication, Postgres storage, and file/object handling while delegating analytics-heavy workloads to scheduled functions as needed.
- **Rationale**: Supabase offers managed auth, database, and storage aligned with mobile-first workflows and accelerates time-to-value for v1.
- **Alternatives Considered**: Custom Postgres + Hasura (higher operational overhead) and Firebase (limited SQL expressiveness for analytics pipelines).

### Charting and Visualization
- **Decision**: Leverage fl_chart with custom performance tuning (data decimation, cached axis labels) to meet the 200 ms rendering target.
- **Rationale**: fl_chart provides flexible line charts, integrates with Flutter, and supports imperative updates necessary for live previews; precomputing polylines keeps renders under budget.
- **Alternatives Considered**: Syncfusion (commercial license considerations) and plotting raw Canvas (longer build time).

### Offline Reliability
- **Decision**: Implement an offline queue backed by Drift tables with exponential backoff and jittered retries triggered by connectivity listeners.
- **Rationale**: Maintains KPI compliance for delayed uploads, enables deterministic replay, and surfaces user-facing status updates.
- **Alternatives Considered**: OS-level background sync only (insufficient control) and in-memory buffers (risk of data loss on crashes).

### AI Assistant Integration
- **Decision**: Provide configurable base URL and model name with token exchange handled via backend-issued short-lived session tokens.
- **Rationale**: Aligns with security requirement to avoid long-lived credentials on the device while keeping the front-end ready for OpenAI-compatible providers.
- **Alternatives Considered**: Embedding static API keys (security risk) or deferring AI entry point (fails roadmap commitment).

## Open Questions
- Finalize caregiver vs. patient data visibility and editing permissions.
- Confirm firmware payload encryption or signing strategy for BLE notifications.
- Determine Supabase region/compliance posture relative to healthcare regulations in target markets.
- Establish AI rate limits and supported model list from backend platform.

## References
- Flutter documentation on handling BLE streams and isolates for computation-heavy workloads.
- Supabase security best practices for mobile clients.
- Performance benchmarks for fl_chart rendering with large datasets.
