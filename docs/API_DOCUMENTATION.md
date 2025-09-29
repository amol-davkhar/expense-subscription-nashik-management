#  API 

## 

API，CRUD、、、、。

**URL:** `http://localhost:3001/api`  
**APIURL:** `http://localhost:3001/api/protected`

### 

1. ****:  `POST /api/auth/login` 
2. ****:  `express-session` ，Cookie ID
3. ****:  `/api/protected/*` 

### 

- `POST /api/auth/login` - （ `username`, `password`）
- `POST /api/auth/logout` - 
- `GET /api/auth/me` - 

### 

 `.env` ：

```bash
# 
SESSION_SECRET=your_random_session_secret

# 
ADMIN_USERNAME=admin
ADMIN_PASSWORD=your_secure_password  # ，

# （）
ADMIN_PASSWORD_HASH=$2a$12$XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
```

### 

- **HTTP-Only Cookie**: IDHTTP-Only Cookie，
- ****: 12
- ****:  `secure`  `sameSite` 
- ****: bcrypt

## 

```json
{
  "error": ""
}
```

：
```json
{
  "data": "",
  "message": ""
}
```

## API

- **** - 
- **** - 
- **** - CRUD
- **** - 、
- **** - 
- **** - 
- **** - 
- **** - 
- **** - 
- **** - CRUD
- **** - CRUD
- **** - （Telegram、Email）
- **** - 

---

## 1. 

### GET /health
API。

```json
{
  "message": "Subscription Management Backend is running!",
  "status": "healthy"
}
```

---

## 2.  (Subscriptions)

#### GET /subscriptions

```json
[
  {
    "id": 1,
    "name": "Netflix",
    "plan": "Premium",
    "billing_cycle": "monthly",
    "next_billing_date": "2025-08-01",
    "last_billing_date": "2025-07-01",
    "amount": 15.99,
    "currency": "USD",
    "payment_method_id": 1,
    "payment_method": {
      "id": 1,
      "value": "creditcard",
      "label": ""
    },
    "start_date": "2024-01-01",
    "status": "active",
    "category_id": 1,
    "category": {
      "id": 1,
      "value": "video",
      "label": ""
    },
    "renewal_type": "auto",
    "notes": "",
    "website": "https://netflix.com",
    "created_at": "2025-01-01T00:00:00.000Z",
    "updated_at": "2025-07-01T00:00:00.000Z"
  }
]
```

#### GET /subscriptions/:id

#### GET /subscriptions/stats/overview

```json
{
  "totalSubscriptions": 15,
  "activeSubscriptions": 12,
  "totalMonthlyAmount": 299.99,
  "averageAmount": 24.99
}
```

#### GET /subscriptions/stats/upcoming-renewals

```json
[
  {
    "id": 1,
    "name": "Netflix",
    "next_billing_date": "2025-07-15",
    "amount": 15.99,
    "currency": "USD"
  }
]
```

#### GET /subscriptions/stats/expired

#### GET /subscriptions/category/:category

#### GET /subscriptions/status/:status

#### GET /subscriptions/search

- `q`: 
- `category`: 
- `status`: 

#### GET /subscriptions/:id/payment-history

#### POST /protected/subscriptions

```json
{
  "name": "Netflix",
  "plan": "Premium",
  "billing_cycle": "monthly",
  "next_billing_date": "2025-08-01",
  "amount": 15.99,
  "currency": "USD",
  "payment_method_id": 1,
  "start_date": "2025-07-01",
  "status": "active",
  "category_id": 1,
  "renewal_type": "auto",
  "notes": "",
  "website": "https://netflix.com"
}
```

#### POST /protected/subscriptions/bulk

#### PUT /protected/subscriptions/:id

#### DELETE /protected/subscriptions/:id

#### POST /protected/subscriptions/reset

## 3.  (Subscription Management)

### POST /protected/subscriptions/auto-renew

### POST /protected/subscriptions/process-expired

### POST /protected/subscriptions/:id/manual-renew

### POST /protected/subscriptions/:id/reactivate

### POST /protected/subscriptions/batch-process

### GET /protected/subscriptions/stats

### GET /protected/subscriptions/upcoming-renewals

## 4.  (Payment History)

#### GET /payment-history

- `subscription_id`: ID
- `start_date`: 
- `end_date`: 
- `limit`: 
- `offset`: 

#### GET /payment-history/:id

#### GET /payment-history/stats/monthly

#### GET /payment-history/stats/yearly

#### GET /payment-history/stats/quarterly

#### POST /protected/payment-history

#### PUT /protected/payment-history/:id

#### DELETE /protected/payment-history/:id

## 5.  (Analytics)

#### GET /analytics/monthly-revenue

#### GET /analytics/monthly-active-subscriptions

#### GET /analytics/revenue-trends

#### GET /analytics/subscription-overview

## 6.  (Settings)

#### GET /settings

#### GET /settings/currencies

#### GET /settings/themes

#### PUT /protected/settings

#### POST /protected/settings/reset

## 7.  (Exchange Rates)

#### GET /exchange-rates

#### GET /exchange-rates/:from/:to

#### GET /exchange-rates/convert

#### POST /protected/exchange-rates

#### POST /protected/exchange-rates/update

## 8. 

###  (Categories)

#### GET /categories

#### POST /protected/categories

#### PUT /protected/categories/:value

#### DELETE /protected/categories/:value

###  (Payment Methods)

#### GET /payment-methods

#### POST /protected/payment-methods

#### PUT /protected/payment-methods/:value

#### DELETE /protected/payment-methods/:value

- `400` - 
- `401` - （）
  - ：`{"error": "Authentication required"}`
  - ：`{"error": "Session expired"}`
- `404` - 
- `500` - 

## Cookie 

API，Cookie：

```javascript
// 
fetch('/api/auth/login', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
  },
  credentials: 'include', // ：Cookie
  body: JSON.stringify({
    username: 'admin',
    password: 'your_password'
  })
});

// 
fetch('/api/protected/subscriptions', {
  method: 'GET',
  credentials: 'include' // ：Cookie
});
```

#### 1. 
```bash
# 
curl -X POST http://localhost:3001/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "admin",
    "password": "your_password"
  }' \
  -c cookie.txt

# 
curl -b cookie.txt http://localhost:3001/api/auth/me
```

#### 2. 
```bash
# 
curl -X POST http://localhost:3001/api/protected/subscriptions \
  -H "Content-Type: application/json" \
  -b cookie.txt \
  -d '{
    "name": "Netflix",
    "plan": "Premium",
    "billing_cycle": "monthly",
    "next_billing_date": "2025-08-01",
    "amount": 15.99,
    "currency": "USD",
    "payment_method_id": 1,
    "category_id": 1,
    "renewal_type": "auto"
  }'

# 
curl -b cookie.txt http://localhost:3001/api/subscriptions

# 
curl -b cookie.txt "http://localhost:3001/api/analytics/monthly-revenue?start_date=2025-01-01&end_date=2025-12-31&currency=USD"
```

#### 3. 
```bash
curl -X POST http://localhost:3001/api/auth/logout \
  -b cookie.txt
```

#### 
```bash
curl -X POST http://localhost:3001/api/protected/notifications/test \
  -H "Content-Type: application/json" \
  -b cookie.txt \
  -d '{"channel_type": "email"}'
```

#### 
```bash
curl -X PUT http://localhost:3001/api/protected/notifications/settings/1 \
  -H "Content-Type: application/json" \
  -b cookie.txt \
  -d '{
    "notification_channels": ["telegram", "email"],
    "is_enabled": true,
    "advance_days": 7
  }'
```

## 9.  (Notifications)

### 
- `renewal_reminder` - 
- `expiration_warning` - 
- `renewal_success` - 
- `renewal_failure` - 
- `subscription_change` - 

### 
- `telegram` - Telegram
- `email` - 

### 

#### GET /notifications/history

- `page` (): ，1
- `limit` (): ，20
- `status` ():  (sent/failed)
- `type` (): 

#### GET /notifications/stats

#### GET /protected/notifications/settings/:userId

#### PUT /protected/notifications/settings/:id

```json
{
  "is_enabled": true,
  "advance_days": 7,
  "notification_channels": ["telegram", "email"],
  "repeat_notification": false
}
```

#### GET /protected/notifications/channels/:channelType

#### POST /protected/notifications/channels

```json
{
  "channel_type": "telegram",
  "config": {
    "chat_id": "123456789"
  }
}
```

#### POST /protected/notifications/test

```json
{
  "channel_type": "email"
}
```

#### POST /protected/notifications/send

```json
{
  "subscription_id": 42,
  "notification_type": "renewal_reminder",
  "channels": ["telegram", "email"]
}
```

#### GET /protected/scheduler/settings

#### PUT /protected/scheduler/settings

```json
{
  "notification_check_time": "09:00",
  "timezone": "Asia/Shanghai",
  "is_enabled": true
}
```

#### POST /protected/scheduler/trigger

```bash
# 
PORT=3001
NODE_ENV=production
SESSION_SECRET=your_random_session_secret

# 
BASE_CURRENCY=CNY
DATABASE_PATH=/app/data/database.sqlite

# 
ADMIN_USERNAME=admin
ADMIN_PASSWORD=your_secure_password  # 
ADMIN_PASSWORD_HASH=$2a$12$...  # 

# API
TIANAPI_KEY=your_tianapi_key

# Telegram
TELEGRAM_BOT_TOKEN=your_telegram_bot_token

# 
EMAIL_HOST=smtp.gmail.com
EMAIL_PORT=587
EMAIL_SECURE=false
EMAIL_USER=your_email@gmail.com
EMAIL_PASSWORD=your_app_password
EMAIL_FROM=Subscription Manager <no-reply@example.com>
```