## Notification System Description

### Overview
- Goal: Provide notifications for subscription management, including expiration reminder, expiration warning, renewal result, and subscription change
- Supported Notification Types:
  - renewal_reminder
  - expiration_warning
  - renewal_success
  - renewal_failure
  - subscription_change
- Supported Channels: telegram, email (SMTP)
- Multilingual: zh-CN, en (Template rendering based on user preferred language)
- Email Service: Based on nodemailer, supports various SMTP providers (Gmail, Outlook, Custom SMTP, etc.)
- Template System: Supports HTML email templates, custom styling and branding

### Architecture and Main Components
- Configuration
  - server/config/notification.js: Notification type, channel, language, timezone, and default values
  - server/config/notificationTemplates.js: Multilingual, multi-channel templates
  - server/config/index.js: Email service configuration (SMTP settings, authentication information, etc.)
- Services
  - NotificationService (server/services/notificationService.js):
    - Unified sending entry point (sendNotification)
    - Send by channel (sendToChannel)
    - Render template (renderMessageTemplate)
    - Record history (createNotificationRecord)
  - TelegramService (server/services/services/telegramService.js): Call Telegram Bot API to send messages
  - EmailService (server/services/emailService.js): Encapsulates nodemailer-based SMTP email sending
    - SMTP configuration management
    - HTML and plain text email support
    - Test email function
    - Error handling and retry mechanism
  - NotificationScheduler (server/services/notificationScheduler.js): cron-based scheduled check and send
- Controllers and Routes
  - NotificationController (server/controllers/notificationController.js): Notification settings, channel configuration, send/test, history, statistics, Telegram utility interfaces
  - Route registration (server/routes/notifications.js, server/routes/scheduler.js; mounted /api and /api/protected in server/server.js)
- Frontend (Partial)
  - src/services/notificationApi.ts: Notification related API client
  - src/components/notification/*: Notification settings UI (TelegramConfig, EmailConfig, NotificationRules, SchedulerSettings, etc.)

### Environment Variables and Runtime Prerequisites

#### Telegram Notification Configuration
- TELEGRAM_BOT_TOKEN: Telegram Bot Token (Required for Telegram notification)

#### Email Notification Configuration
- EMAIL_HOST: SMTP Server Host (e.g., smtp.gmail.com)
- EMAIL_PORT: SMTP Server Port (e.g., 587 for Gmail, 465 for secure)
- EMAIL_SECURE: Whether to use SSL/TLS (Boolean, usually false for Gmail)
- EMAIL_USER: SMTP Authentication Username (Usually the email address)
- EMAIL_PASSWORD: SMTP Authentication Password or App-specific password
- EMAIL_FROM: Default Sender Address (Format: `Subscription Manager <no-reply@example.com>`)
- EMAIL_TLS_REJECT_UNAUTHORIZED: Whether to verify SSL certificate (Default true)
- EMAIL_LOCALE: Localization language for email test message (Default zh-CN)

#### Notification System Configuration
- NOTIFICATION_DEFAULT_CHANNELS: Default notification channels JSON string (Default ["telegram"])
- NOTIFICATION_DEFAULT_LANGUAGE: Default notification language (Default zh-CN)

#### Authentication Configuration (For accessing protected interfaces)
- SESSION_SECRET: Session key (Required)
- ADMIN_USERNAME: Administrator username (Default admin)
- ADMIN_PASSWORD: Administrator password (Used at first launch)
- ADMIN_PASSWORD_HASH: Administrator password hash (Recommended for production environment)

Note: All protected interfaces require session-based login authentication. Email notification requires configuring valid SMTP server information.

### Database Structure (Key Tables)

#### notification_settings (Notification Settings Table)
- id: INTEGER PRIMARY KEY AUTOINCREMENT
- notification_type: TEXT NOT NULL UNIQUE (Renewal reminder, expiration warning, etc.)
- is_enabled: BOOLEAN NOT NULL DEFAULT 1 (Whether enabled)
- advance_days: INTEGER DEFAULT 7 (Advance days)
- repeat_notification: BOOLEAN NOT NULL DEFAULT 0 (Whether to repeat reminder)
- notification_channels: TEXT NOT NULL DEFAULT '["telegram"]' (JSON array, supports multiple channels)
- created_at: DATETIME DEFAULT CURRENT_TIMESTAMP
- updated_at: DATETIME DEFAULT CURRENT_TIMESTAMP

#### notification_channels (Notification Channel Configuration Table)
- id: INTEGER PRIMARY KEY AUTOINCREMENT
- channel_type: TEXT NOT NULL UNIQUE (telegram/email)
- channel_config: TEXT NOT NULL (JSON format configuration)
  - Telegram: `{"chat_id": "123456789"}`
  - Email: `{"email": "user@example.com"}`
- is_active: BOOLEAN NOT NULL DEFAULT 1 (Whether active)
- last_used_at: DATETIME (Last used time)
- created_at: DATETIME DEFAULT CURRENT_TIMESTAMP
- updated_at: DATETIME DEFAULT CURRENT_TIMESTAMP

#### notification_history (Notification History Table)
- id: INTEGER PRIMARY KEY AUTOINCREMENT
- subscription_id: INTEGER NOT NULL (Associated subscription)
- notification_type: TEXT NOT NULL (Notification type)
- channel_type: TEXT NOT NULL (Notification channel)
- status: TEXT NOT NULL CHECK (status IN ('sent', 'failed')) (Sending status)
- recipient: TEXT NOT NULL (Recipient identifier)
- message_content: TEXT NOT NULL (Message content)
- error_message: TEXT (Error message, recorded upon sending failure)
- sent_at: DATETIME (Sent time)
- created_at: DATETIME DEFAULT CURRENT_TIMESTAMP
- FOREIGN KEY (subscription_id) REFERENCES subscriptions (id) ON DELETE CASCADE

#### scheduler_settings (Scheduler Settings Table)
- id: INTEGER PRIMARY KEY CHECK (id = 1)
- notification_check_time: TEXT NOT NULL DEFAULT '09:00' (Check time HH:mm)
- timezone: TEXT NOT NULL DEFAULT 'Asia/Shanghai' (Timezone)
- is_enabled: BOOLEAN NOT NULL DEFAULT 1 (Whether to enable scheduling)
- created_at: DATETIME DEFAULT CURRENT_TIMESTAMP
- updated_at: DATETIME DEFAULT CURRENT_TIMESTAMP

Note: Specific table creation statements, indexes, and triggers can be referenced in the server/db/migrations.js file.

### Templates and Multilingual Support
- Templates are located in server/config/notificationTemplates.js, organized by notification type/language/channel.
- If no template is found, it falls back to a short text generated by NotificationService.getDefaultContent.
- Template variable examples: name, plan, amount, currency, payment_method, next_billing_date, billing_cycle, etc.

Available template utility interfaces (read-only):
- GET /api/templates/languages
- GET /api/templates/types
- GET /api/templates/channels?notificationType=...&language=...
- GET /api/templates/template?notificationType=...&language=...&channel=...
- GET /api/templates/overview

### API Overview (/api/protected requires login)

- Notification Settings (Protected: /api/protected/notifications)
  - GET /settings/:userId
  - GET /settings/:userId/:type
  - PUT /settings/:id
    - body: { is_enabled, advance_days, notification_channels, repeat_notification }
    - Note: When notification_type is expiration_warning, advance_days will be enforced as 0

- Channel Configuration (Protected: /api/protected/notifications)
  - POST /channels
    - body: { channel_type: "telegram"|"email"|"webhook", config: { ... } }
  - GET /channels/:channelType

- Send and Test (Protected: /api/protected/notifications)
  - POST /send
    - body: { user_id?, subscription_id, notification_type, channels? }
    - If channels is not provided, send according to configuration/default channels
  - POST /test
    - body: { channel_type }

- History and Statistics (Public: /api/notifications)
  - GET /history?page=&limit=&status=&type=
  - GET /stats

- Telegram Utility (Protected: /api/protected/notifications)
  - POST /validate-chat-id (body: { chat_id })
  - GET /telegram/bot-info
  - GET /telegram/config-status

- Scheduler
  - Public (/api/scheduler)
    - GET /settings
    - GET /status
  - Protected (/api/protected/scheduler)
    - PUT /settings (Update notification check time, timezone, switch)
    - POST /trigger (Manually trigger a check)

Request Examples (curl):

- Update Notification Settings
  curl -X PUT \
       -H "Content-Type: application/json" \
       -b cookie.txt -c cookie.txt \
       -d '{"is_enabled":true,"advance_days":7,"notification_channels":["telegram"],"repeat_notification":false}' \
       http://localhost:3001/api/protected/notifications/settings/1

- Configure Telegram Channel
  curl -X POST \
       -H "Content-Type: application/json" \
       -b cookie.txt -c cookie.txt \
       -d '{"channel_type":"telegram","config":{"chat_id":"123456789"}}' \
       http://localhost:3001/api/protected/notifications/channels

- Send Test Notification
  curl -X POST \
       -H "Content-Type: application/json" \
       -b cookie.txt -c cookie.txt \
       -d '{"channel_type":"telegram"}' \
       http://localhost:3001/api/protected/notifications/test

- Manually Send Notification
  curl -X POST \
       -H "Content-Type: application/json" \
       -b cookie.txt -c cookie.txt \
       -d '{"subscription_id":42,"notification_type":"renewal_reminder","channels":["telegram"]}' \
       http://localhost:3001/api/protected/notifications/send

- Update Scheduler Settings
  curl -X PUT \
       -H "Content-Type: application/json" \
       -b cookie.txt -c cookie.txt \
       -d '{"notification_check_time":"09:00","timezone":"Asia/Shanghai","is_enabled":true}' \
       http://localhost:3001/api/protected/scheduler/settings

### Sending Process (Server Logic Key Points)
1) Validate if the notification type is supported
2) Read notification settings (Enabled status, advance days, channels, etc.)
3) Load subscription data (subscription_id)
4) Select template based on user language preference and render content (Use default text if no template is found)
5) Send in parallel according to enabled channels (Currently supports telegram)
6) Write result to notification_history (sent/failed, timestamp, error message)

### Scheduler Behavior
- Configure daily check time and timezone (HH:mm) in scheduler_settings, executed by node-cron expression in the corresponding timezone
- checkAndSendNotifications will scan subscriptions requiring reminder/warning and call sendNotification for sending

### Telegram Configuration Guide
- Configure TELEGRAM_BOT_TOKEN environment variable
- Validate chat_id legality via /api/protected/notifications/validate-chat-id
- Save chat_id configuration via /api/protected/notifications/channels
- Use /api/protected/notifications/test for channel connectivity test

### Email Notification Configuration Guide

#### 1. SMTP Server Configuration
Configure the following environment variables in the `.env` file:

```bash
# Gmail SMTP Configuration Example
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_SECURE=false
EMAIL_USER=your_email@gmail.com
EMAIL_PASSWORD=your_app_password  # Gmail requires an App-specific password
EMAIL_FROM=Subscription Manager <no-reply@example.com>
EMAIL_LOCALE=zh-CN

# Outlook SMTP Configuration Example
EMAIL_HOST=smtp-mail.outlook.com
EMAIL_PORT=587
EMAIL_SECURE=false
EMAIL_USER=your_email@outlook.com
EMAIL_PASSWORD=your_password
EMAIL_FROM=Subscription Manager <no-reply@outlook.com>
```

#### 2. Gmail
1. Gmail
2. Google
3. EMAIL_PASSWORD

#### 3. Configure Email Channel
```bash
# Configure email channel via API
curl -X POST \
  -H "Content-Type: application/json" \
  -b cookie.txt -c cookie.txt \
  -d '{"channel_type":"email","config":{"email":"your_email@example.com"}}' \
  http://localhost:3001/api/protected/notifications/channels
```

#### 4. Test Email Sending
```bash
# Send test email
curl -X POST \
  -H "Content-Type: application/json" \
  -b cookie.txt -c cookie.txt \
  -d '{"channel_type":"email"}' \
  http://localhost:3001/api/protected/notifications/test
```

### Reference Files

#### Backend Core Files
- server/config/notification.js - Notification system configuration
- server/config/notificationTemplates.js - Notification template definition
- server/config/index.js - Email service configuration
- server/services/notificationService.js - Notification service main class
- server/services/notificationScheduler.js - Notification scheduler
- server/services/telegramService.js - Telegram service
- server/services/emailService.js - Email service
- server/controllers/notificationController.js - Notification controller
- server/routes/notifications.js - Notification routes
- server/routes/scheduler.js - Scheduler routes
- server/server.js - Server entry point and route registration

#### Frontend Files
- src/services/notificationApi.ts - Notification API client
- src/components/notification/* - Notification settings UI components
- EmailConfig.tsx - Email configuration component
- NotificationHistory.tsx - Notification history component
- NotificationRules.tsx - Notification rules component
- NotificationSettings.tsx - Notification settings component

#### Database Related
- server/db/migrations.js - Database migration script
- server/db/schema.sql - Database structure definition