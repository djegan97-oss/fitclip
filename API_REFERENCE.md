# FitClip API Reference

**Version:** 1.0.0  
**Base URL:** `https://api.fitclip.golf/v1`  
**Last Updated:** November 28, 2025  
**OpenAPI Spec:** `https://api.fitclip.golf/v1/openapi.json`

---

## TABLE OF CONTENTS

1. [Authentication](#authentication)
2. [Common Patterns](#common-patterns)
3. [Error Handling](#error-handling)
4. [Rate Limiting](#rate-limiting)
5. [Endpoints](#endpoints)
   - [Auth](#auth-endpoints)
   - [Shops](#shop-endpoints)
   - [Fittings](#fitting-endpoints)
   - [Shots](#shot-endpoints)
   - [Videos](#video-endpoints)
   - [Customers](#customer-endpoints)
   - [TrackMan](#trackman-endpoints)
   - [Analytics](#analytics-endpoints)
   - [Webhooks](#webhook-endpoints)

---

## AUTHENTICATION

### Overview

FitClip API uses **JWT (JSON Web Tokens)** for authentication. All requests (except auth endpoints) require a valid access token.

### Token Types

1. **Access Token:** Short-lived (15 minutes), used for API requests
2. **Refresh Token:** Long-lived (30 days), used to obtain new access tokens

### Authentication Flow

```
1. POST /auth/login
   { "email": "owner@shop.com", "password": "password" }
   
2. Response:
   {
     "access_token": "eyJhbGciOiJIUzI1NiIs...",
     "refresh_token": "eyJhbGciOiJIUzI1NiIs...",
     "expires_in": 900
   }

3. Store tokens securely:
   - Access token: In-memory (React state)
   - Refresh token: httpOnly cookie (secure, SameSite=Strict)

4. Include access token in all requests:
   Authorization: Bearer eyJhbGciOiJIUzI1NiIs...

5. When access token expires (401 error):
   POST /auth/refresh
   { "refresh_token": "..." }
   Returns new access token
```

### Request Headers

```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

---

## COMMON PATTERNS

### Pagination

List endpoints support cursor-based pagination:

```http
GET /v1/fittings?limit=20&cursor=eyJpZCI6ImFiYzEyMyJ9
```

**Response:**
```json
{
  "data": [...],
  "pagination": {
    "next_cursor": "eyJpZCI6ImRlZjQ1NiJ9",
    "has_more": true
  }
}
```

### Filtering

Use query parameters for filtering:

```http
GET /v1/fittings?shop_id=abc123&status=completed&date_from=2025-11-01
```

### Sorting

```http
GET /v1/fittings?sort_by=completed_at&sort_order=desc
```

### Field Selection

Request only needed fields:

```http
GET /v1/fittings/abc123?fields=id,customer_name,status
```

---

## ERROR HANDLING

### Error Response Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request parameters",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address"
      }
    ]
  },
  "request_id": "req_abc123xyz"
}
```

### Error Codes

| HTTP Status | Error Code | Description |
|-------------|------------|-------------|
| 400 | VALIDATION_ERROR | Request parameters invalid |
| 401 | UNAUTHORIZED | Missing or invalid access token |
| 403 | FORBIDDEN | Insufficient permissions |
| 404 | NOT_FOUND | Resource not found |
| 409 | CONFLICT | Resource already exists |
| 422 | UNPROCESSABLE_ENTITY | Valid request but business logic error |
| 429 | RATE_LIMIT_EXCEEDED | Too many requests |
| 500 | INTERNAL_SERVER_ERROR | Server error |
| 503 | SERVICE_UNAVAILABLE | Temporarily unavailable |

---

## RATE LIMITING

**Limits:**
- **Per Shop:** 1,000 requests per 15 minutes
- **Per IP:** 100 requests per minute (unauthenticated)

**Headers:**
```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 987
X-RateLimit-Reset: 1701345678
```

**When Exceeded:**
```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Retry after 15 minutes.",
    "retry_after": 900
  }
}
```

---

## ENDPOINTS

## AUTH ENDPOINTS

### POST /auth/login

Authenticate user and receive tokens.

**Request:**
```json
{
  "email": "owner@precisiongolf.com",
  "password": "SecurePassword123!"
}
```

**Response:** `200 OK`
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 900,
  "user": {
    "id": "usr_abc123",
    "email": "owner@precisiongolf.com",
    "name": "Mike Chen",
    "role": "shop_owner",
    "shop_id": "shp_def456"
  }
}
```

**Errors:**
- `401 INVALID_CREDENTIALS` - Email or password incorrect
- `403 ACCOUNT_DISABLED` - Account has been disabled

---

### POST /auth/refresh

Obtain new access token using refresh token.

**Request:**
```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response:** `200 OK`
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 900
}
```

**Errors:**
- `401 INVALID_REFRESH_TOKEN` - Token expired or invalid

---

### POST /auth/logout

Invalidate refresh token.

**Request:**
```json
{
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

**Response:** `204 No Content`

---

### POST /auth/reset-password

Request password reset email.

**Request:**
```json
{
  "email": "owner@precisiongolf.com"
}
```

**Response:** `200 OK`
```json
{
  "message": "Password reset email sent"
}
```

**Note:** Always returns 200 even if email doesn't exist (security best practice)

---

### POST /auth/verify-email

Verify email address with token from email.

**Request:**
```json
{
  "token": "eml_verify_abc123xyz"
}
```

**Response:** `200 OK`
```json
{
  "message": "Email verified successfully"
}
```

---

## SHOP ENDPOINTS

### POST /shops

Create new shop (onboarding).

**Request:**
```json
{
  "name": "Precision Golf Lab",
  "email": "hello@precisiongolf.com",
  "phone": "+14155551234",
  "address": {
    "street": "123 Golf Drive",
    "city": "Austin",
    "state": "TX",
    "zip": "78701",
    "country": "US"
  },
  "owner": {
    "first_name": "Mike",
    "last_name": "Chen",
    "email": "mike@precisiongolf.com",
    "password": "SecurePassword123!"
  },
  "subscription_tier": "unlimited"
}
```

**Response:** `201 Created`
```json
{
  "id": "shp_abc123",
  "name": "Precision Golf Lab",
  "email": "hello@precisiongolf.com",
  "phone": "+14155551234",
  "address": { ... },
  "subscription": {
    "tier": "unlimited",
    "status": "trialing",
    "trial_ends_at": "2025-12-28T00:00:00Z",
    "price_per_month": 19900
  },
  "created_at": "2025-11-28T14:30:00Z",
  "onboarding_completed": false
}
```

**Errors:**
- `409 EMAIL_EXISTS` - Shop with this email already exists
- `400 VALIDATION_ERROR` - Invalid phone number or address

---

### GET /shops/:id

Get shop details.

**Response:** `200 OK`
```json
{
  "id": "shp_abc123",
  "name": "Precision Golf Lab",
  "email": "hello@precisiongolf.com",
  "phone": "+14155551234",
  "address": { ... },
  "branding": {
    "logo_url": "https://cdn.fitclip.golf/assets/shp_abc123/logo.png",
    "primary_color": "#1B4332",
    "secondary_color": "#FF6B35"
  },
  "subscription": {
    "tier": "unlimited",
    "status": "active",
    "mrr": 19900,
    "next_billing_date": "2025-12-28T00:00:00Z"
  },
  "statistics": {
    "total_fittings": 1247,
    "total_videos": 1247,
    "videos_this_month": 103,
    "conversion_rate": 0.68
  },
  "created_at": "2025-08-15T10:00:00Z",
  "updated_at": "2025-11-28T14:30:00Z"
}
```

---

### PATCH /shops/:id

Update shop details.

**Request:**
```json
{
  "name": "Precision Golf Lab - Downtown",
  "phone": "+14155559999",
  "branding": {
    "primary_color": "#2A5A3F"
  }
}
```

**Response:** `200 OK`
```json
{
  "id": "shp_abc123",
  "name": "Precision Golf Lab - Downtown",
  // ... updated fields
  "updated_at": "2025-11-28T15:00:00Z"
}
```

---

### POST /shops/:id/logo

Upload shop logo.

**Request:** `multipart/form-data`
```
Content-Type: multipart/form-data
file: logo.png (max 5 MB, PNG or JPG)
```

**Response:** `200 OK`
```json
{
  "logo_url": "https://cdn.fitclip.golf/assets/shp_abc123/logo.png?v=2",
  "uploaded_at": "2025-11-28T15:10:00Z"
}
```

**Errors:**
- `400 FILE_TOO_LARGE` - File exceeds 5 MB
- `400 INVALID_FILE_TYPE` - Must be PNG or JPG

---

### GET /shops/:id/analytics

Get shop performance analytics.

**Query Parameters:**
- `date_from` (ISO 8601 date): Start date
- `date_to` (ISO 8601 date): End date
- `metrics` (comma-separated): Specific metrics to return

**Response:** `200 OK`
```json
{
  "shop_id": "shp_abc123",
  "date_range": {
    "from": "2025-11-01T00:00:00Z",
    "to": "2025-11-28T23:59:59Z"
  },
  "metrics": {
    "fittings": {
      "total": 103,
      "completed": 98,
      "abandoned": 5
    },
    "videos": {
      "generated": 98,
      "watched": 90,
      "watch_rate": 0.92,
      "avg_watch_duration_seconds": 165,
      "shared": 68,
      "share_rate": 0.76
    },
    "conversions": {
      "purchases": 67,
      "conversion_rate": 0.68,
      "avg_order_value": 1380.00,
      "total_revenue": 92460.00
    },
    "performance_by_fitter": [
      {
        "fitter_name": "Sarah Martinez",
        "fittings": 42,
        "conversion_rate": 0.74,
        "avg_order_value": 1420.00
      },
      {
        "fitter_name": "John Smith",
        "fittings": 56,
        "conversion_rate": 0.64,
        "avg_order_value": 1340.00
      }
    ]
  }
}
```

---

## FITTING ENDPOINTS

### POST /fittings

Create new fitting session.

**Request:**
```json
{
  "shop_id": "shp_abc123",
  "bay_id": "bay_def456",
  "customer": {
    "first_name": "Tom",
    "last_name": "Rodriguez",
    "email": "tom@example.com",
    "phone": "+14155558888",
    "handicap": 12.0
  },
  "fitting_type": "driver",
  "fitter_name": "Sarah Martinez"
}
```

**Response:** `201 Created`
```json
{
  "id": "fit_xyz789",
  "shop_id": "shp_abc123",
  "customer_id": "cust_hij012",
  "customer": {
    "first_name": "Tom",
    "last_name": "Rodriguez",
    "email": "tom@example.com",
    "phone": "+14155558888"
  },
  "fitting_type": "driver",
  "fitter_name": "Sarah Martinez",
  "status": "in_progress",
  "started_at": "2025-11-28T14:30:00Z",
  "shot_count": 0
}
```

---

### GET /fittings

List fittings for shop.

**Query Parameters:**
- `shop_id` (required): Shop ID
- `status`: Filter by status (in_progress, completed, video_generated)
- `date_from`, `date_to`: Date range
- `limit`: Results per page (default 20, max 100)
- `cursor`: Pagination cursor

**Response:** `200 OK`
```json
{
  "data": [
    {
      "id": "fit_xyz789",
      "customer_name": "Tom Rodriguez",
      "fitting_type": "driver",
      "fitter_name": "Sarah Martinez",
      "status": "completed",
      "shot_count": 28,
      "started_at": "2025-11-28T14:30:00Z",
      "completed_at": "2025-11-28T15:45:00Z"
    },
    // ... more fittings
  ],
  "pagination": {
    "next_cursor": "eyJpZCI6ImZpdF9hYmMxMjMifQ==",
    "has_more": true
  }
}
```

---

### GET /fittings/:id

Get fitting details.

**Response:** `200 OK`
```json
{
  "id": "fit_xyz789",
  "shop_id": "shp_abc123",
  "customer": {
    "id": "cust_hij012",
    "first_name": "Tom",
    "last_name": "Rodriguez",
    "email": "tom@example.com",
    "handicap": 12.0
  },
  "fitting_type": "driver",
  "fitter_name": "Sarah Martinez",
  "status": "video_generated",
  "started_at": "2025-11-28T14:30:00Z",
  "completed_at": "2025-11-28T15:45:00Z",
  "shot_count": 28,
  "video": {
    "id": "vid_klm345",
    "status": "ready",
    "url": "https://cdn.fitclip.golf/videos/fit_xyz789/recap.mp4",
    "thumbnail_url": "https://cdn.fitclip.golf/videos/fit_xyz789/thumbnail.jpg",
    "duration_seconds": 180,
    "generated_at": "2025-11-28T16:10:00Z"
  },
  "shots": [
    {
      "id": "shot_001",
      "club_type": "driver",
      "club_brand": "TaylorMade",
      "club_model": "Stealth 2",
      "ball_speed_mph": 145.2,
      "carry_distance_yards": 245.3,
      "is_highlight": true,
      "highlight_position": 1
    },
    // ... 27 more shots
  ]
}
```

---

### PATCH /fittings/:id/complete

Complete fitting and trigger video generation.

**Request:**
```json
{
  "shot_ids_to_exclude": ["shot_001", "shot_002"],
  "fitter_notes": "Customer loved TaylorMade Stealth driver. Recommended Ventus Blue shaft."
}
```

**Response:** `200 OK`
```json
{
  "id": "fit_xyz789",
  "status": "completed",
  "completed_at": "2025-11-28T15:45:00Z",
  "shot_count": 26,
  "video_job": {
    "job_id": "job_nop678",
    "status": "queued",
    "estimated_completion": "2025-11-28T16:15:00Z"
  }
}
```

**Video Generation Trigger:**
- Job queued in Redis (Celery)
- Worker picks up job within 30 seconds
- Video ready in 15-30 minutes
- Customer receives SMS + email when complete

---

### DELETE /fittings/:id

Delete fitting (soft delete).

**Response:** `204 No Content`

**Note:** Soft delete (status = 'deleted', data retained for 30 days)

---

## SHOT ENDPOINTS

### POST /fittings/:fitting_id/shots

Add shot to fitting (called by FitClip Agent during session).

**Request:**
```json
{
  "trackman_shot_id": "TM_20251128_143022_001",
  "timestamp": "2025-11-28T14:30:22.123Z",
  "club_type": "driver",
  "club_brand": "TaylorMade",
  "club_model": "Stealth 2",
  "club_loft": 10.5,
  "shaft_type": "Ventus Blue 6X",
  "ball_speed_mph": 145.2,
  "club_speed_mph": 98.5,
  "smash_factor": 1.47,
  "launch_angle_deg": 14.2,
  "spin_rate_rpm": 2650,
  "carry_distance_yards": 245.3,
  "total_distance_yards": 262.1,
  "offline_distance_yards": -8.5,
  "apex_height_yards": 32.1,
  "is_baseline": false
}
```

**Response:** `201 Created`
```json
{
  "id": "shot_qrs901",
  "fitting_id": "fit_xyz789",
  "trackman_shot_id": "TM_20251128_143022_001",
  "timestamp": "2025-11-28T14:30:22.123Z",
  "club_type": "driver",
  "ball_speed_mph": 145.2,
  "carry_distance_yards": 245.3,
  "is_valid": true,
  "highlight_score": 87.5,
  "created_at": "2025-11-28T14:30:24.456Z"
}
```

**Validation:**
- Duplicate `trackman_shot_id` returns `409 CONFLICT`
- Invalid shot (ball_speed <50 mph) flagged as `is_valid = false`

---

### GET /fittings/:fitting_id/shots

Get all shots for fitting.

**Query Parameters:**
- `is_valid`: Filter by validity (true, false)
- `is_highlight`: Filter highlights only
- `sort_by`: Sort field (timestamp, carry_distance, highlight_score)
- `sort_order`: asc or desc

**Response:** `200 OK`
```json
{
  "fitting_id": "fit_xyz789",
  "total_shots": 28,
  "valid_shots": 26,
  "invalid_shots": 2,
  "shots": [
    {
      "id": "shot_qrs901",
      "club_type": "driver",
      "ball_speed_mph": 145.2,
      "carry_distance_yards": 245.3,
      "is_valid": true,
      "is_highlight": true,
      "highlight_position": 1,
      "timestamp": "2025-11-28T14:30:22.123Z"
    },
    // ... more shots
  ]
}
```

---

### PATCH /shots/:id

Update shot (e.g., fitter marks as valid/invalid).

**Request:**
```json
{
  "is_valid": true,
  "override_reason": "Felt great despite low smash factor"
}
```

**Response:** `200 OK`
```json
{
  "id": "shot_qrs901",
  "is_valid": true,
  "manual_override": true,
  "override_reason": "Felt great despite low smash factor",
  "updated_at": "2025-11-28T14:35:00Z"
}
```

---

## VIDEO ENDPOINTS

### GET /videos/:id

Get video details.

**Response:** `200 OK`
```json
{
  "id": "vid_klm345",
  "fitting_id": "fit_xyz789",
  "status": "ready",
  "url": "https://cdn.fitclip.golf/videos/fit_xyz789/recap.mp4?token=...",
  "thumbnail_url": "https://cdn.fitclip.golf/videos/fit_xyz789/thumbnail.jpg",
  "duration_seconds": 180,
  "file_size_mb": 42.3,
  "generated_at": "2025-11-28T16:10:00Z",
  "sent_at": "2025-11-28T16:12:00Z",
  "analytics": {
    "view_count": 3,
    "first_viewed_at": "2025-11-28T16:45:00Z",
    "last_viewed_at": "2025-11-29T09:20:00Z",
    "total_watch_duration_seconds": 495,
    "avg_watch_duration_seconds": 165,
    "watch_completion_rate": 0.92,
    "shared_count": 1,
    "purchase_clicked": true,
    "purchase_completed": true
  }
}
```

**Note:** URL includes signed token (expires in 7 days)

---

### POST /videos/:id/regenerate

Regenerate video (admin only, if generation failed).

**Request:**
```json
{
  "reason": "Original generation failed"
}
```

**Response:** `202 Accepted`
```json
{
  "job_id": "job_tuv234",
  "status": "queued",
  "estimated_completion": "2025-11-28T17:00:00Z"
}
```

---

### DELETE /videos/:id

Delete video (customer request, GDPR compliance).

**Response:** `204 No Content`

**Effect:**
- Video deleted from S3 immediately
- Database record soft-deleted
- Customer notified via email

---

## CUSTOMER ENDPOINTS

### POST /customers

Create customer (usually auto-created during fitting).

**Request:**
```json
{
  "shop_id": "shp_abc123",
  "first_name": "Tom",
  "last_name": "Rodriguez",
  "email": "tom@example.com",
  "phone": "+14155558888",
  "handicap": 12.0
}
```

**Response:** `201 Created`
```json
{
  "id": "cust_hij012",
  "shop_id": "shp_abc123",
  "first_name": "Tom",
  "last_name": "Rodriguez",
  "email": "tom@example.com",
  "phone": "+14155558888",
  "handicap": 12.0,
  "total_fittings": 0,
  "created_at": "2025-11-28T14:25:00Z"
}
```

---

### GET /customers/:id

Get customer details.

**Response:** `200 OK`
```json
{
  "id": "cust_hij012",
  "first_name": "Tom",
  "last_name": "Rodriguez",
  "email": "tom@example.com",
  "phone": "+14155558888",
  "handicap": 12.0,
  "total_fittings": 3,
  "total_purchases": 2,
  "lifetime_value": 2760.00,
  "created_at": "2024-03-15T10:00:00Z",
  "fittings": [
    {
      "id": "fit_xyz789",
      "fitting_type": "driver",
      "completed_at": "2025-11-28T15:45:00Z",
      "video_url": "https://..."
    },
    // ... more fittings
  ]
}
```

---

## TRACKMAN ENDPOINTS

### POST /trackman/connect

Connect TrackMan launch monitor to FitClip.

**Request:**
```json
{
  "shop_id": "shp_abc123",
  "bay_label": "Bay 1 - Main Shop",
  "trackman_model": "trackman_4",
  "username": "shop@trackman.com",
  "api_key": "TM_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}
```

**Response:** `201 Created`
```json
{
  "id": "tm_wxy567",
  "shop_id": "shp_abc123",
  "bay_label": "Bay 1 - Main Shop",
  "trackman_model": "trackman_4",
  "serial_number": "TM-123456",
  "status": "connected",
  "last_poll_at": "2025-11-28T14:30:00Z",
  "expires_at": "2025-12-28T14:30:00Z",
  "created_at": "2025-11-28T14:30:00Z"
}
```

**Errors:**
- `401 INVALID_CREDENTIALS` - TrackMan API key invalid
- `422 TRACKMAN_UNREACHABLE` - Cannot connect to TrackMan

---

### GET /trackman/:id/status

Check TrackMan connection status.

**Response:** `200 OK`
```json
{
  "id": "tm_wxy567",
  "status": "connected",
  "last_poll_at": "2025-11-28T14:30:22Z",
  "last_shot_at": "2025-11-28T14:30:22Z",
  "shots_today": 142,
  "token_expires_at": "2025-12-28T14:30:00Z"
}
```

**Status Values:**
- `connected` - Active, polling
- `disconnected` - Network or auth issue
- `expired` - Token expired, needs refresh

---

### DELETE /trackman/:id

Disconnect TrackMan.

**Response:** `204 No Content`

---

## ANALYTICS ENDPOINTS

### POST /analytics/track

Track analytics event (video view, button click, purchase).

**Request:**
```json
{
  "event_type": "video_viewed",
  "video_id": "vid_klm345",
  "customer_id": "cust_hij012",
  "shop_id": "shp_abc123",
  "timestamp": "2025-11-28T16:45:00Z",
  "metadata": {
    "watch_duration_seconds": 165,
    "device": "mobile",
    "referrer": "sms"
  }
}
```

**Response:** `202 Accepted`
```json
{
  "event_id": "evt_zab890",
  "status": "queued"
}
```

**Event Types:**
- `video_viewed` - Customer watched video
- `video_shared` - Customer shared on social
- `purchase_clicked` - Customer clicked "Buy Now"
- `purchase_completed` - Customer completed purchase
- `email_opened` - Customer opened email
- `sms_clicked` - Customer clicked SMS link

---

### GET /analytics/report

Generate analytics report.

**Query Parameters:**
- `shop_id` (required)
- `date_from`, `date_to`
- `report_type`: daily, weekly, monthly
- `metrics`: Comma-separated list

**Response:** `200 OK`
```json
{
  "shop_id": "shp_abc123",
  "report_type": "monthly",
  "date_range": {
    "from": "2025-11-01",
    "to": "2025-11-30"
  },
  "summary": {
    "fittings": 103,
    "videos_generated": 98,
    "videos_watched": 90,
    "purchases": 67,
    "conversion_rate": 0.68,
    "revenue": 92460.00
  },
  "daily_breakdown": [
    {
      "date": "2025-11-01",
      "fittings": 4,
      "videos_watched": 3,
      "purchases": 2
    },
    // ... more days
  ]
}
```

---

## WEBHOOK ENDPOINTS

### POST /webhooks/stripe

Stripe webhook for subscription events.

**Headers:**
```http
Stripe-Signature: t=1701234567,v1=abc123...
```

**Request:** (Stripe webhook payload)
```json
{
  "type": "invoice.payment_succeeded",
  "data": {
    "object": {
      "customer": "cus_abc123",
      "amount_paid": 19900
    }
  }
}
```

**Response:** `200 OK`

**Event Types Handled:**
- `invoice.payment_succeeded` - Subscription payment
- `invoice.payment_failed` - Payment failed
- `customer.subscription.deleted` - Subscription canceled

---

### POST /webhooks/trackman

TrackMan webhook (if supported in future).

**Request:**
```json
{
  "event": "shot_captured",
  "serial_number": "TM-123456",
  "shot_data": { ... }
}
```

**Response:** `200 OK`

---

## OPENAPI SPECIFICATION

Full OpenAPI 3.0 spec available at:

```
https://api.fitclip.golf/v1/openapi.json
```

Import into Postman, Insomnia, or generate client SDKs using:

```bash
openapi-generator-cli generate -i openapi.json -g typescript-axios -o ./client
```

---

## SDK & LIBRARIES

### Official SDKs

**JavaScript/TypeScript:**
```bash
npm install @fitclip/sdk
```

```typescript
import { FitClipClient } from '@fitclip/sdk';

const client = new FitClipClient({ apiKey: 'your_access_token' });

const fitting = await client.fittings.create({
  shop_id: 'shp_abc123',
  customer: { ... }
});
```

**Python:**
```bash
pip install fitclip-python
```

```python
from fitclip import FitClipClient

client = FitClipClient(api_key='your_access_token')

fitting = client.fittings.create(
    shop_id='shp_abc123',
    customer={...}
)
```

---

## POSTMAN COLLECTION

Import ready-to-use Postman collection:

```
https://api.fitclip.golf/v1/postman-collection.json
```

---

## SUPPORT

**API Issues:**
- Email: api-support@fitclip.golf
- Slack: [FitClip Developer Community](https://fitclip-dev.slack.com)
- Status Page: https://status.fitclip.golf

**Documentation:**
- Developer Portal: https://developers.fitclip.golf
- Guides & Tutorials: https://docs.fitclip.golf

---

**API Reference v1.0.0**  
© 2025 FitClip Technologies, Inc.

