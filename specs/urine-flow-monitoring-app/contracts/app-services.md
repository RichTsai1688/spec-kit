# Application Service Contracts

## AuthenticationService
- **Responsibilities**: Sign in/out, refresh tokens, expose current user session, exchange tokens for AI requests.
- **Interface**:
  - `Future<UserSession> signIn(String email, String password)`
  - `Future<void> signOut()`
  - `Future<UserSession> refreshSession()`
  - `Future<String> requestAiToken()`

## DevicePairingService
- **Responsibilities**: Discover devices, pair/unpair, manage connection lifecycle, send commands.
- **Interface**:
  - `Stream<List<DeviceSummary>> watchNearbyDevices()`
  - `Future<void> pairDevice(DeviceSummary device)`
  - `Future<void> unpairDevice(String deviceId)`
  - `Future<void> startMeasurement(SessionSeed seed)`
  - `Stream<MeasurementSample> measurementStream()`

## MeasurementService
- **Responsibilities**: Drive state machine, buffer samples, compute metrics, coordinate upload queue.
- **Interface**:
  - `Future<MeasurementSession> createSession(Device device)`
  - `Stream<MeasurementState> watchState(String sessionId)`
  - `Future<void> finalizeSession(String sessionId, MeasurementSummary summary)`
  - `Future<void> abortSession(String sessionId, String reason)`

## MetricsCalculator
- **Responsibilities**: Derive Qmax, Qavg, voided volume, time to peak, flow time from samples.
- **Interface**:
  - `MeasurementSummary compute(List<MeasurementSample> samples)`

## HistoryRepository
- **Responsibilities**: Persist sessions, expose filtering, produce trend aggregates.
- **Interface**:
  - `Stream<List<SessionSummary>> watchRecentSessions(Duration range)`
  - `Future<TrendSnapshot> loadTrend(TrendRange range)`
  - `Future<SessionDetail> loadDetail(String sessionId)`

## UploadQueueManager
- **Responsibilities**: Manage offline queue, retry policies, ack handling.
- **Interface**:
  - `Future<void> enqueue(UploadPayload payload)`
  - `Stream<List<UploadPayload>> watchPending()`
  - `Future<void> acknowledge(String ackToken)`
  - `Future<void> retryPending()`

## AiAdvisorService
- **Responsibilities**: Build prompt context, call configurable endpoint, handle streaming responses.
- **Interface**:
  - `Future<AiResponse> askQuestion(String prompt, List<MetricsSnapshot> context)`
  - `void setConfiguration(AiConfiguration config)`

## ObservabilityService
- **Responsibilities**: Capture logs, metrics, and crash reports while respecting privacy constraints.
- **Interface**:
  - `void logEvent(String name, Map<String, Object?> properties)`
  - `void recordError(Object error, StackTrace stackTrace)`
