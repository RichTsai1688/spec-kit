# Quickstart: Urine Flow Monitoring Mobile App

This guide helps contributors run the mobile application, simulate a measurement, and validate end-to-end data flow.

## Prerequisites
- Flutter SDK 3.24+ with Dart 3.x (`flutter --version` to verify)
- Xcode 15+ (for iOS) and Android Studio / Android SDK 34+ (for Android)
- Supabase project with Auth and Postgres enabled; service role key stored securely
- BLE urine flow sensor or simulator supporting the `urineflow-svc` service UUID
- Node.js 18+ for running Supabase CLI or local tooling (optional)

## Environment Setup
1. Clone the repository and install Flutter dependencies:
   ```bash
   git clone <REPO_URL>
   cd <REPO_DIR>
   flutter pub get
   ```
2. Create `.env` files under `mobile/` for staging and production containing:
   ```env
   SUPABASE_URL=...
   SUPABASE_ANON_KEY=...
   AI_BASE_URL=https://api.example.com
   AI_MODEL=gpt-4o
   ```
3. Configure Supabase:
   - Enable email/password authentication.
   - Create tables matching `data-model.md` (users, devices, sessions, samples, metrics, upload_queue).
   - Set up Row Level Security to restrict access to owner accounts.
4. Provision BLE simulator (if hardware unavailable):
   - Use a development board or mobile BLE simulator to broadcast the `urineflow-svc` UUID with command and stream characteristics.
   - Ensure the simulator responds to the `START{session_key,...}` command and emits 20-50 Hz sample notifications.

## Running the App
1. Launch a Supabase local dev instance or connect to staging.
2. From the repo root, run the Flutter app:
   ```bash
   flutter run -d ios   # or -d android
   ```
3. Sign in with a test account created in Supabase.
4. Pair the BLE simulator or hardware device through the pairing screen.
5. Tap **Start Measurement** and observe:
   - Countdown begins and device receives the start command.
   - Real-time curve updates with <1 second latency.
   - Session concludes after 20 seconds and shows computed metrics.
   - Upload status transitions to "Synced" in history view.

## Smoke Tests
- **Authentication**: Sign out and back in to confirm session persistence and secure storage of tokens.
- **Offline Queue**: Disable network after measurement completion; verify upload queue stores pending payloads and retries once network returns.
- **Trend Visualization**: Record three sessions, open weekly trend page, and confirm sparkline rendering within the performance budget.
- **AI Entry Point**: Configure AI settings, open the assistant screen, and confirm the payload preview includes latest metrics.

## Troubleshooting
- If BLE pairing fails, reset the simulator and clear the app's pairing cache.
- When chart rendering exceeds 200 ms, check for debug mode overhead and confirm data decimation is active.
- For Supabase connection issues, verify network access and that anon keys are current; rotate credentials if compromised.
