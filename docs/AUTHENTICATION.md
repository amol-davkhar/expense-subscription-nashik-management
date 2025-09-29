# Authentication Overview

This document explains how authentication works in the subscription management system after migrating from API key checks to session-based login.

## High-Level Design

- **Session-Based Authentication**: The backend uses `express-session` to issue an **HTTP-only** Cookie upon successful login. This Cookie only stores the **session ID**; user data is kept in the server-side **session store** (which defaults to memory storage).
- **Single Administrator Account**: An administrator user is configured via **environment variables**. All requests sent to `/api/**` or `/api/protected/**` must come from an **authenticated session**.
- **Frontend Protection**: The **React application** fetches the current session (`GET /api/auth/me`) on startup and **blocks all protected routes** until the check is complete.
- **Notification System Integration**: All notification-related configuration and management features are protected by the same **session authentication system**, ensuring only authenticated users can configure and use the notification functionality.

## Configuration

Set the following **environment variables** in the `.env` file (backend root or project root, depending on deployment method):

SESSION_SECRET=your_random_session_secret
ADMIN_USERNAME=admin
ADMIN_PASSWORD=your_secure_password

When the server starts, a **bcrypt hash** is derived from `ADMIN_PASSWORD`, logged once, and saved in memory. Copy the generated value into `ADMIN_PASSWORD_HASH` and **delete `ADMIN_PASSWORD`** for **production deployment**.

Other notes:

- `SESSION_SECRET` must be a **long random string** to ensure session Cookies cannot be forged. If missing, the backend generates a **temporary key** for each process and prints a warning; this causes **session invalidation** on every restart, so explicit setting is advised.
- `ADMIN_PASSWORD_HASH` (if provided) takes **precedence** over the plaintext password. It should be a **bcrypt hash** generated with a cost of $\ge 12$.
- **Key Rotation**: Updating `ADMIN_USERNAME` or `ADMIN_PASSWORD_HASH` requires a **server restart**. Existing sessions remain valid until they expire.
- `TRUST_PROXY`: Needed when the backend is deployed behind a **reverse proxy**, **load balancer**, or **CDN** (e.g., Nginx, Caddy, Cloudflare). Set the proxy level (e.g., `1` for a single-layer proxy). Without it, a `secure` Cookie may not take effect, causing the browser to discard the `sid`.
- `SESSION_COOKIE_SECURE`: Controls the `cookie.secure` behavior of `express-session`. Defaults to `auto` (automatically enabled in production). Can be explicitly set to `true` or `false` when in an **HTTP intranet** or when overriding the default behavior is required.
- `SESSION_COOKIE_SAMESITE`: Controls the **SameSite** policy (`lax`/`strict`/`none`). If the frontend accesses from a **cross-site HTTPS domain**, it must be set to `none` and paired with `SESSION_COOKIE_SECURE=true`.


## SESSION_SECRET and ADMIN_PASSWORD_HASH Generation Methods

### SESSION_SECRET Generation Method

- It is recommended to use a **high-strength random string** with a length of at least 32 bytes.
- Can be generated using the following command:

```bash
openssl rand -base64 48
```

#### ADMIN_PASSWORD_HASH

1. `.env`, `ADMIN_PASSWORD`：

   ```bash
   ADMIN_PASSWORD=your_secure_password
   ```

2. `ADMIN_PASSWORD_HASH`

3. `.env`, `ADMIN_PASSWORD_HASH`, `ADMIN_PASSWORD`

- Node.js 示例：

  ```js
  // bcryptjs
  npm install -g bcryptjs
  npx bcryptjs your_secure_password 12
  ```

```
SESSION_SECRET=change_me_to_a_long_random_string
ADMIN_USERNAME=admin
ADMIN_PASSWORD=your_secure_password
```

Solidified for Production (Recommended)

```
SESSION_SECRET=long_random_string_generated_once
ADMIN_USERNAME=admin
ADMIN_PASSWORD_HASH=$2a$12$XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
# Delete ADMIN_PASSWORD
```