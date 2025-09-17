# Data Model: Urine Flow Monitoring Mobile App

## Entity Overview

### User
- **Description**: Represents patients, caregivers, or support personnel accessing the application.
- **Attributes**:
  - `id`: globally unique identifier
  - `role`: patient | caregiver | support
  - `email`: login credential
  - `hashed_uid`: hashed identifier shared with devices
  - `created_at`: account creation timestamp
  - `status`: active | suspended | deleted
  - `preferences`: localization and notification settings
- **Relationships**: One-to-many with Devices and Sessions; may have delegated access to other user accounts (caregiver binding pending clarification).

### Device
- **Description**: Physical urine flow monitoring sensor paired with the app.
- **Attributes**:
  - `id`: internal identifier
  - `serial_no`: hardware serial number
  - `pair_user_id`: reference to owning user
  - `comm_type`: BLE | WIFI_MQTT
  - `firmware_version`
  - `last_seen_at`
  - `status`: paired | unpaired | maintenance
- **Relationships**: Belongs to a User; produces Sessions.

### Session
- **Description**: Measurement run initiated from the mobile app.
- **Attributes**:
  - `id`: unique identifier
  - `session_key`: generated token shared with device and cloud
  - `user_id`: foreign key to User
  - `device_id`: foreign key to Device
  - `start_ts`: session start timestamp (UTC)
  - `duration_s`: expected duration (default 20)
  - `status`: running | completed | aborted | failed | upload_pending
  - `notes`: optional clinician/patient notes
- **Relationships**: Has many Samples and one Metrics record.

### Sample
- **Description**: Individual telemetry record captured during a session.
- **Attributes**:
  - `id`: unique identifier
  - `session_id`: foreign key to Session
  - `t_ms`: elapsed milliseconds from session start
  - `flow_ml_s`: instantaneous flow rate
  - `quality`: indicator (e.g., good, interpolated, dropped)
- **Relationships**: Belongs to Session; aggregated into Metrics.

### Metric Summary
- **Description**: Computed statistics summarizing a Session.
- **Attributes**:
  - `session_id`: primary key referencing Session
  - `q_max`: maximum flow rate
  - `q_avg`: average flow rate
  - `voided_volume_est`: estimated voided volume
  - `time_to_peak`: milliseconds to peak flow
  - `flow_time_s`: effective flow duration
  - `duration_s`: actual measurement length
  - `tq_curve_hash`: hash for verifying curve integrity
  - `quality_flags`: aggregate quality indicators
- **Relationships**: One-to-one with Session.

### Upload Queue Item
- **Description**: Tracks pending uploads when offline.
- **Attributes**:
  - `id`: unique identifier
  - `session_id`: reference to Session
  - `payload_type`: session_metadata | samples_batch | metrics
  - `retry_count`: number of retry attempts
  - `next_retry_at`: scheduled retry timestamp
  - `last_error`: most recent error message/code
- **Relationships**: Belongs to Session; consumed by sync worker.

## Derived Data & Views
- **Weekly Trend View**: Aggregates sessions per week with median metrics and sparkline-ready downsampled curves.
- **Monthly Distribution View**: Calculates quartiles for Qmax/Qavg and voided volume for box-plot visualizations.
- **Quality Audit View**: Flags sessions exceeding packet loss thresholds or containing long gaps for maintenance review.

## Validation Rules
- Session duration must equal 20 seconds unless aborted; aborted sessions require status notes.
- Sample `flow_ml_s` must remain within hardware-provided min/max bounds; values outside range trigger quality flag.
- Metrics must be recomputed whenever samples are modified or re-uploaded.
- Upload queue items expire after successful acknowledgement from the cloud backend.
