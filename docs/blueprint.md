# Password Distribution Bot — Bot specification

**Archetype:** workflow

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot for secure password distribution: admins upload CSV files with student credentials, review validation results, and send passwords via private messages after confirmation. Tracks delivery status and provides audit logs.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- school administrators
- IT staff

## Success criteria

- Admin receives confirmation summary after all passwords are sent
- Each student receives private message with their credentials
- All delivery statuses are logged and visible to admins

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Open main menu with upload option
- **/upload** (command, actor: user, command: /upload) — Initiate CSV upload wizard
- **Confirm Send** (button, actor: user, callback: batch:confirm) — Finalize password distribution after review
- **Cancel Batch** (button, actor: user, callback: batch:cancel) — Abort current upload process

## Flows

### Password Distribution Workflow
_Trigger:_ /upload

1. Admin uploads CSV file
2. Bot validates format and content
3. Admin reviews validation results
4. Admin confirms or cancels
5. Bot maps usernames to Telegram accounts
6. Bot sends private messages
7. Bot generates delivery report

_Data touched:_ StudentRecord, UploadBatch, AuditLogEntry

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **StudentRecord** _(retention: persistent)_ — Individual student credential delivery record
  - fields: username, password, telegram_user_id, status, delivery_timestamp
- **UploadBatch** _(retention: persistent)_ — CSV upload metadata and validation results
  - fields: file_hash, uploader_id, timestamp, row_count, error_rows
- **AuditLogEntry** _(retention: persistent)_ — Action history for compliance and troubleshooting
  - fields: admin_id, action_type, target_batch, outcome

## Integrations

- **Telegram** (required) — Private message delivery and admin notifications
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- CSV file format validation rules
- Telegram username mapping policy
- Message template customization
- Batch retention period configuration

## Notifications

- Delivery success/failure summaries to admin
- Error notifications for unresolved usernames
- Final audit report after batch completion

## Permissions & privacy

- Telegram account access limited to verified students
- Password data stored encrypted with 90-day retention
- Admin actions fully audited

## Edge cases

- Unresolvable Telegram usernames in CSV
- Invalid CSV format with missing headers
- Duplicate username entries
- Telegram message delivery failures

## Required tests

- End-to-end CSV upload to delivery workflow
- Error handling for invalid inputs
- Message encryption and retention policies
- Audit log completeness verification

## Assumptions

- CSV contains exactly username/password columns
- Telegram usernames map directly to student records
- 90-day retention is sufficient for compliance
