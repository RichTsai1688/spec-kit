# Tasks: Urine Flow Monitoring Mobile App

**Input**: Design documents from `/specs/urine-flow-monitoring-app/`
**Prerequisites**: plan.md (required), research.md, data-model.md, contracts/

## Phase 3.1: Setup
- [ ] T001 Scaffold Flutter project in `mobile/` with package structure and CI-friendly `analysis_options.yaml`.
- [ ] T002 Configure `mobile/pubspec.yaml` with Riverpod, go_router, dio, flutter_blue_plus, mqtt_client, fl_chart, Drift, integration_test, and dotenv support.
- [ ] T003 Establish environment configuration loader in `mobile/lib/app/config/app_config.dart` and secure storage service shell.
- [ ] T004 [P] Create shared theme, typography, and reusable card/chart container widgets in `mobile/lib/app/theme/` and `mobile/lib/widgets/`.

## Phase 3.2: Tests First (TDD)
- [ ] T005 Author widget test for authentication flow in `mobile/test/features/authentication/login_flow_test.dart`.
- [ ] T006 [P] Write integration test simulating BLE measurement lifecycle in `mobile/integration_test/measurement_stream_test.dart` using mock transports.
- [ ] T007 [P] Create integration test for history trend aggregation in `mobile/integration_test/history_trend_test.dart` seeded with Drift fixtures.
- [ ] T008 [P] Draft unit tests for metric calculations (Qmax, Qavg, volume, TTP) in `mobile/test/features/measurement/metrics_calculator_test.dart`.
- [ ] T009 [P] Add contract test for AI payload builder in `mobile/test/features/ai_advisor/ai_request_builder_test.dart`.

## Phase 3.3: Core Implementation
- [ ] T010 Implement Riverpod providers and routing for authentication, measurement, history, and AI features in `mobile/lib/app/`.
- [ ] T011 Build authentication screens and Supabase service integration in `mobile/lib/features/authentication/` with secure token handling.
- [ ] T012 Develop device pairing module with BLE discovery, connection, and session start command composer in `mobile/lib/features/pairing/`.
- [ ] T013 Implement measurement state machine, countdown UI, and live chart rendering in `mobile/lib/features/measurement/` meeting latency targets.
- [ ] T014 Construct data buffering layer and ring buffer handling with quality flag propagation in `mobile/lib/infrastructure/streaming/`.
- [ ] T015 Persist sessions, samples, metrics, and upload queue using Drift DAOs in `mobile/lib/infrastructure/persistence/`.
- [ ] T016 Build statistics summary view and result screen in `mobile/lib/features/measurement_result/` including share/upload status states.
- [ ] T017 Implement history views (weekly/monthly filters, sparklines, comparisons) in `mobile/lib/features/history/`.
- [ ] T018 Create AI advisor UI shell with configurable base URL/model and context packaging in `mobile/lib/features/ai_advisor/`.

## Phase 3.4: Integration
- [ ] T019 Integrate Supabase APIs for session upload, samples batch transfer, and ACK handling in `mobile/lib/infrastructure/networking/` with retry policies.
- [ ] T020 Connect offline queue processor with connectivity listeners and exponential backoff scheduler in `mobile/lib/infrastructure/sync/`.
- [ ] T021 Implement secure storage of refreshed tokens and session binding validation in `mobile/lib/infrastructure/security/`.
- [ ] T022 Wire telemetry logging and crash reporting hooks in `mobile/lib/app/observability/`.

## Phase 3.5: Polish
- [ ] T023 [P] Expand unit tests for error handling and retry logic across services in `mobile/test/`.
- [ ] T024 [P] Add performance benchmarks for chart rendering and metrics computation in `mobile/test/performance/`.
- [ ] T025 Prepare privacy policy screen and account deletion flow in `mobile/lib/features/settings/privacy_page.dart`.
- [ ] T026 Implement beta build configurations and release pipelines (TestFlight, Play Internal) documented in `docs/release-checklist.md`.
- [ ] T027 Conduct manual QA checklist covering measurement, history, AI entry point, and offline recovery in `docs/manual-testing.md`.

## Dependencies
- Tests (T005-T009) must be implemented and failing before corresponding features (T010-T018).
- T012 (pairing) and T014 (streaming) block T013 (measurement UI) and T019 (upload integration).
- T015 (persistence) is prerequisite for history (T017) and offline queue (T020).
- T019 and T020 must complete before beta packaging (T026) to validate end-to-end reliability.

## Parallel Example
```
# Parallelize metric-related efforts once persistence is ready:
Task: "T008 Draft unit tests for metric calculations..."
Task: "T016 Build statistics summary view..."
Task: "T024 Add performance benchmarks for chart rendering..."
```

## Notes
- Maintain separation between infrastructure adapters (BLE, MQTT, Supabase) and domain services to simplify testing.
- Keep AI advisor feature guarded behind configuration toggles until backend tokens are available.
- Ensure offline queue includes crash-safe checkpoints to uphold <0.5% data loss objective.
