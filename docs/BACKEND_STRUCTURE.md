# Backend Architecture & Database Structure
# RentKart — India's Premier Rental Marketplace
**Version:** 1.0 | **Pattern:** RESTful API | **ORM:** Prisma + PostgreSQL 16.2

---

## 1. Architecture Overview

- **Pattern:** RESTful API (JSON over HTTPS)
- **Authentication Strategy:** JWT (access token 15min + refresh token 30 days stored in Redis)
- **Data Flow:** Client → Fastify API → Zod Validation → Business Logic → Prisma ORM → PostgreSQL
- **Caching Strategy:** Redis for: OTP (60s TTL), session/refresh tokens, rate-limit counters, order state pub/sub, homepage collection caches (5 min TTL)
- **File Handling:** AWS S3 for all media (listing photos, inspection photos, KYC documents); CloudFront CDN for delivery
- **Payment Flow:** Client → Razorpay (PA-licensed) → webhook → RentKart escrow logic

---

## 2. Database Schema

---

### Table: `users`
**Purpose:** All user accounts — buyers, sellers, delivery agents, admins

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PRIMARY KEY, DEFAULT gen_random_uuid() | Unique identifier |
| phone | VARCHAR(13) | UNIQUE, NOT NULL | Indian mobile number with country code |
| phone_verified | BOOLEAN | DEFAULT FALSE | OTP verification status |
| password_hash | VARCHAR(255) | NOT NULL | bcrypt hash (12 rounds) |
| full_name | VARCHAR(255) | NOT NULL | User display name |
| role | VARCHAR(20) | NOT NULL, CHECK (role IN ('buyer','seller','admin','delivery')) | User type |
| email | VARCHAR(255) | UNIQUE, NULLABLE | Optional email |
| profile_photo_url | VARCHAR(500) | NULLABLE | S3 URL |
| city | VARCHAR(100) | NULLABLE | Primary city |
| is_active | BOOLEAN | DEFAULT TRUE | Account active status |
| is_verified | BOOLEAN | DEFAULT FALSE | KYC/admin verified |
| device_fingerprint | VARCHAR(500) | NULLABLE | For fraud detection |
| aadhaar_id_hash | VARCHAR(255) | UNIQUE, NULLABLE | Hashed Aadhaar ID for blacklist checks |
| trustworthiness_score | INTEGER | DEFAULT 0 | Accumulated trust score (buyer) |
| created_at | TIMESTAMP | DEFAULT NOW() | |
| updated_at | TIMESTAMP | DEFAULT NOW() | |

**Indexes:**
- `idx_users_phone` ON (phone)
- `idx_users_role` ON (role)
- `idx_users_device_fingerprint` ON (device_fingerprint)
- `idx_users_aadhaar_hash` ON (aadhaar_id_hash)

---

### Table: `kyc_documents`
**Purpose:** KYC submission records for sellers (and buyers for high-value orders)

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | |
| user_id | UUID | FK → users(id) ON DELETE CASCADE, NOT NULL | Owner |
| doc_type | VARCHAR(30) | NOT NULL, CHECK (doc_type IN ('aadhaar','pan','address_proof')) | Document type |
| doc_url | VARCHAR(500) | NOT NULL | Encrypted S3 URL |
| status | VARCHAR(20) | DEFAULT 'pending', CHECK (status IN ('pending','approved','rejected')) | Review status |
| rejection_reason | TEXT | NULLABLE | Admin rejection note |
| reviewed_by | UUID | FK → users(id) ON DELETE SET NULL, NULLABLE | Admin who reviewed |
| reviewed_at | TIMESTAMP | NULLABLE | |
| created_at | TIMESTAMP | DEFAULT NOW() | |
| updated_at | TIMESTAMP | DEFAULT NOW() | |

---

### Table: `seller_profiles`
**Purpose:** Extended profile data for seller accounts

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | |
| user_id | UUID | FK → users(id) ON DELETE CASCADE, UNIQUE, NOT NULL | One-to-one with users |
| store_name | VARCHAR(255) | NOT NULL | |
| store_bio | TEXT | NULLABLE | |
| store_logo_url | VARCHAR(500) | NULLABLE | S3 URL |
| delivery_mode | VARCHAR(20) | DEFAULT 'platform', CHECK (delivery_mode IN ('platform','self')) | |
| bank_account_number | VARCHAR(20) | NULLABLE | Encrypted at rest |
| bank_ifsc | VARCHAR(11) | NULLABLE | |
| bank_verified | BOOLEAN | DEFAULT FALSE | Penny drop status |
| subscription_plan | VARCHAR(20) | DEFAULT 'free', CHECK (subscription_plan IN ('free','premium')) | |
| subscription_expires_at | TIMESTAMP | NULLABLE | |
| commission_rate | DECIMAL(4,3) | DEFAULT 0.150 | Override per seller (premium = 0.120) |
| avg_rating | DECIMAL(3,2) | DEFAULT 0.00 | Materialised from reviews |
| total_rentals | INTEGER | DEFAULT 0 | Materialised count |
| cancellation_count | INTEGER | DEFAULT 0 | 60-day rolling count |
| created_at | TIMESTAMP | DEFAULT NOW() | |
| updated_at | TIMESTAMP | DEFAULT NOW() | |

---

### Table: `listings`
**Purpose:** Items listed by sellers for rent

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | |
| seller_id | UUID | FK → users(id) ON DELETE CASCADE, NOT NULL | Listing owner |
| title | VARCHAR(80) | NOT NULL | |
| description | TEXT | NOT NULL, CHECK (char_length(description) >= 100) | |
| category_l1 | VARCHAR(50) | NOT NULL | Top-level category |
| category_l2 | VARCHAR(50) | NOT NULL | Sub-category |
| condition | VARCHAR(20) | NOT NULL, CHECK (condition IN ('new','like_new','gently_used','used')) | |
| item_value | INTEGER | NOT NULL, CHECK (item_value >= 500 AND item_value <= 500000) | Full MRP in paise (store as integer ₹) |
| daily_rate_pct | DECIMAL(4,3) | NOT NULL, CHECK (daily_rate_pct >= 0.005 AND daily_rate_pct <= 0.05) | e.g. 0.020 = 2% |
| location_pincode | VARCHAR(6) | NOT NULL | |
| city | VARCHAR(100) | NOT NULL | |
| brand | VARCHAR(100) | NULLABLE | |
| size_details | JSONB | NULLABLE | {chest, waist, bust, length} |
| occasion_tags | VARCHAR(30)[] | NULLABLE | Array of tags |
| care_instructions | TEXT | NULLABLE | |
| inventory_count | INTEGER | DEFAULT 1, CHECK (inventory_count >= 1) | Multiple identical units |
| status | VARCHAR(20) | DEFAULT 'draft', CHECK (status IN ('draft','pending_review','live','under_cleaning','archived')) | |
| qr_tag_id | VARCHAR(50) | UNIQUE, NULLABLE | Physical QR tag code sewn into item |
| is_boosted | BOOLEAN | DEFAULT FALSE | Featured listing boost |
| boost_expires_at | TIMESTAMP | NULLABLE | |
| admin_approved_at | TIMESTAMP | NULLABLE | When admin approved this listing |
| seller_listing_count | INTEGER | DEFAULT 0 | Denormalised; reset from query |
| created_at | TIMESTAMP | DEFAULT NOW() | |
| updated_at | TIMESTAMP | DEFAULT NOW() | |

**Indexes:**
- `idx_listings_seller_id` ON (seller_id)
- `idx_listings_status` ON (status)
- `idx_listings_city_category` ON (city, category_l1, category_l2)
- `idx_listings_is_boosted` ON (is_boosted) WHERE is_boosted = TRUE
- GIN index for full-text search: `idx_listings_search` ON (to_tsvector('english', title || ' ' || description))

---

### Table: `listing_photos`
**Purpose:** Photos and video associated with a listing

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | |
| listing_id | UUID | FK → listings(id) ON DELETE CASCADE, NOT NULL | |
| url | VARCHAR(500) | NOT NULL | S3 URL |
| type | VARCHAR(10) | DEFAULT 'photo', CHECK (type IN ('photo','video')) | |
| sort_order | INTEGER | DEFAULT 0 | Display order |
| created_at | TIMESTAMP | DEFAULT NOW() | |

**Indexes:**
- `idx_listing_photos_listing_id` ON (listing_id)

---

### Table: `listing_availability`
**Purpose:** Availability blocks and blackout dates per listing

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | |
| listing_id | UUID | FK → listings(id) ON DELETE CASCADE, NOT NULL | |
| block_type | VARCHAR(20) | NOT NULL, CHECK (block_type IN ('booked','blackout','cleaning')) | |
| start_date | DATE | NOT NULL | |
| end_date | DATE | NOT NULL | |
| order_id | UUID | FK → orders(id) ON DELETE SET NULL, NULLABLE | If blocked by an order |
| created_at | TIMESTAMP | DEFAULT NOW() | |

---

### Table: `orders`
**Purpose:** Core rental order records

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | |
| order_number | VARCHAR(20) | UNIQUE, NOT NULL | Human-readable (RK-2024-XXXXX) |
| buyer_id | UUID | FK → users(id), NOT NULL | |
| seller_id | UUID | FK → users(id), NOT NULL | |
| listing_id | UUID | FK → listings(id), NOT NULL | |
| delivery_agent_id | UUID | FK → users(id) ON DELETE SET NULL, NULLABLE | Assigned agent |
| status | VARCHAR(30) | NOT NULL | See order status lifecycle |
| delivery_address | JSONB | NOT NULL | {line1, line2, city, state, pincode} |
| start_date | DATE | NOT NULL | Rental start (delivery date) |
| end_date | DATE | NULLABLE | Return scheduled date |
| actual_delivery_date | DATE | NULLABLE | When agent marks delivered |
| actual_return_date | DATE | NULLABLE | When agent marks returned |
| auto_sale_date | DATE | NOT NULL | start_date + 25 |
| item_value | INTEGER | NOT NULL | Snapshot from listing at time of order (₹) |
| daily_rate_pct | DECIMAL(4,3) | NOT NULL | Snapshot from listing |
| delivery_charge | INTEGER | NOT NULL | In ₹ |
| total_paid | INTEGER | NOT NULL | item_value + delivery_charge (₹) |
| rental_days | INTEGER | NULLABLE | Calculated on return |
| usage_charge | INTEGER | NULLABLE | Calculated on return (₹) |
| damage_penalty | INTEGER | DEFAULT 0 | (₹) |
| late_fee | INTEGER | DEFAULT 0 | (₹) |
| platform_commission | INTEGER | NULLABLE | usage_charge × commission_rate (₹) |
| buyer_refund | INTEGER | NULLABLE | item_value - usage_charge - damage_penalty - late_fee (₹) |
| seller_payout | INTEGER | NULLABLE | usage_charge - platform_commission (₹) |
| refund_status | VARCHAR(20) | DEFAULT 'pending' | pending / processing / completed / failed |
| payout_status | VARCHAR(20) | DEFAULT 'pending' | pending / processing / completed / failed |
| refund_razorpay_id | VARCHAR(100) | NULLABLE | Razorpay refund ID |
| payout_razorpay_id | VARCHAR(100) | NULLABLE | Razorpay payout ID |
| auto_sale_triggered | BOOLEAN | DEFAULT FALSE | |
| created_at | TIMESTAMP | DEFAULT NOW() | |
| updated_at | TIMESTAMP | DEFAULT NOW() | |

**Order Status Lifecycle:**
```
confirmed → dispatched → delivered → in_use → return_scheduled
→ return_picked_up → returned → inspected → refund_issued
                                           → converted_to_sale (Day 25)
                                           → disputed
```

**Indexes:**
- `idx_orders_buyer_id` ON (buyer_id)
- `idx_orders_seller_id` ON (seller_id)
- `idx_orders_status` ON (status)
- `idx_orders_auto_sale_date` ON (auto_sale_date) WHERE auto_sale_triggered = FALSE
- `idx_orders_listing_id` ON (listing_id)

---

### Table: `order_inspections`
**Purpose:** Photo evidence captured at delivery and return by delivery agent

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | |
| order_id | UUID | FK → orders(id) ON DELETE CASCADE, NOT NULL | |
| inspection_type | VARCHAR(10) | NOT NULL, CHECK (inspection_type IN ('delivery','return')) | |
| captured_by | UUID | FK → users(id), NOT NULL | Delivery agent |
| photos | JSONB | NOT NULL | Array of {url, angle_label, gps_lat, gps_lng, timestamp} |
| qr_tag_scanned | VARCHAR(50) | NULLABLE | QR code scanned at this inspection |
| qr_tag_match | BOOLEAN | NULLABLE | Does scanned tag match listing.qr_tag_id |
| damage_level | VARCHAR(10) | DEFAULT 'none', CHECK (damage_level IN ('none','minor','moderate','severe')) | Agent classification |
| damage_notes | TEXT | NULLABLE | Agent written notes |
| buyer_otp_confirmed | BOOLEAN | DEFAULT FALSE | Delivery OTP confirmation |
| seller_review_completed | BOOLEAN | DEFAULT FALSE | Seller reviewed within 24h window |
| created_at | TIMESTAMP | DEFAULT NOW() | |
| updated_at | TIMESTAMP | DEFAULT NOW() | |

---

### Table: `disputes`
**Purpose:** Dispute tickets raised by buyer or seller

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | |
| order_id | UUID | FK → orders(id), NOT NULL | |
| raised_by | UUID | FK → users(id), NOT NULL | |
| dispute_type | VARCHAR(30) | NOT NULL, CHECK (dispute_type IN ('damage_dispute','wrong_item','non_return','other')) | |
| description | TEXT | NOT NULL | |
| evidence_urls | VARCHAR(500)[] | NULLABLE | Additional photos |
| status | VARCHAR(20) | DEFAULT 'open', CHECK (status IN ('open','under_review','resolved','closed')) | |
| resolved_by | UUID | FK → users(id) ON DELETE SET NULL, NULLABLE | Admin |
| resolution_type | VARCHAR(30) | NULLABLE | e.g. full_refund / partial_refund / penalty_applied / no_action |
| resolution_notes | TEXT | NULLABLE | Admin decision notes |
| resolved_at | TIMESTAMP | NULLABLE | |
| created_at | TIMESTAMP | DEFAULT NOW() | |
| updated_at | TIMESTAMP | DEFAULT NOW() | |

---

### Table: `reviews`
**Purpose:** Post-rental buyer reviews for listings and sellers

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | |
| order_id | UUID | FK → orders(id) ON DELETE CASCADE, UNIQUE, NOT NULL | One review per order |
| reviewer_id | UUID | FK → users(id), NOT NULL | Buyer |
| listing_id | UUID | FK → listings(id), NOT NULL | |
| seller_id | UUID | FK → users(id), NOT NULL | |
| item_rating | SMALLINT | NOT NULL, CHECK (item_rating BETWEEN 1 AND 5) | |
| seller_rating | SMALLINT | NOT NULL, CHECK (seller_rating BETWEEN 1 AND 5) | |
| review_text | TEXT | NULLABLE | |
| photo_urls | VARCHAR(500)[] | NULLABLE | Up to 5 review photos |
| is_verified | BOOLEAN | DEFAULT TRUE | Always true — only from completed orders |
| is_flagged | BOOLEAN | DEFAULT FALSE | Seller-flagged for manipulation review |
| created_at | TIMESTAMP | DEFAULT NOW() | |
| updated_at | TIMESTAMP | DEFAULT NOW() | |

---

### Table: `payments`
**Purpose:** Payment transaction records (linked to Razorpay)

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | |
| order_id | UUID | FK → orders(id), NOT NULL | |
| razorpay_order_id | VARCHAR(100) | UNIQUE, NOT NULL | Razorpay order ID |
| razorpay_payment_id | VARCHAR(100) | UNIQUE, NULLABLE | Filled on capture |
| amount | INTEGER | NOT NULL | In paise (₹ × 100) |
| currency | VARCHAR(3) | DEFAULT 'INR' | |
| status | VARCHAR(20) | DEFAULT 'created' | created / captured / failed / refunded |
| method | VARCHAR(20) | NULLABLE | upi / card / netbanking / wallet |
| webhook_payload | JSONB | NULLABLE | Raw Razorpay webhook |
| created_at | TIMESTAMP | DEFAULT NOW() | |
| updated_at | TIMESTAMP | DEFAULT NOW() | |

---

### Table: `notifications`
**Purpose:** Record of all push/SMS/email notifications sent

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | |
| user_id | UUID | FK → users(id) ON DELETE CASCADE, NOT NULL | |
| type | VARCHAR(50) | NOT NULL | e.g. order_confirmed, day_20_reminder |
| channel | VARCHAR(10) | NOT NULL, CHECK (channel IN ('push','sms','email')) | |
| title | VARCHAR(200) | NULLABLE | Push notification title |
| body | TEXT | NOT NULL | Message content |
| status | VARCHAR(20) | DEFAULT 'sent' | sent / delivered / failed |
| created_at | TIMESTAMP | DEFAULT NOW() | |

---

### Table: `admin_logs`
**Purpose:** Audit trail for all admin actions

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | UUID | PRIMARY KEY | |
| admin_id | UUID | FK → users(id), NOT NULL | Admin who acted |
| action | VARCHAR(100) | NOT NULL | e.g. approve_kyc, refund_triggered |
| entity_type | VARCHAR(50) | NOT NULL | user / listing / order / dispute |
| entity_id | UUID | NOT NULL | ID of affected entity |
| details | JSONB | NULLABLE | Action parameters/context |
| created_at | TIMESTAMP | DEFAULT NOW() | |

---

## 3. API Endpoints

---

### POST /api/auth/register
**Purpose:** Create new user account
**Authentication:** Public

**Request Body:**
```json
{
  "phone": "9876543210",
  "password": "SecurePass123!",
  "full_name": "Priya Sharma",
  "role": "buyer",
  "city": "Pune"
}
```

**Validation:**
- `phone`: 10-digit Indian mobile number, unique in DB
- `password`: 8+ chars, at least 1 uppercase, 1 lowercase, 1 number, 1 special char
- `full_name`: 2–255 chars
- `role`: must be 'buyer' or 'seller' (admin/delivery set by admin only)

**Response (201):**
```json
{
  "message": "Account created. OTP sent to your mobile.",
  "userId": "uuid"
}
```

**Errors:**
- 400: Validation failed (field-level details)
- 409: Phone number already registered

---

### POST /api/auth/verify-otp
**Purpose:** Verify phone OTP after registration or login
**Authentication:** Public

**Request Body:**
```json
{ "phone": "9876543210", "otp": "482931" }
```

**Response (200):**
```json
{
  "accessToken": "eyJ...",
  "refreshToken": "eyJ...",
  "user": { "id": "uuid", "fullName": "Priya Sharma", "role": "buyer", "isVerified": false }
}
```

**Errors:**
- 400: Invalid or expired OTP (max 5 attempts; 30-min lockout after)
- 404: Phone number not found

---

### POST /api/auth/login
**Purpose:** Login via phone + password
**Authentication:** Public

**Request Body:**
```json
{ "phone": "9876543210", "password": "SecurePass123!" }
```

**Response (200):**
```json
{
  "accessToken": "eyJ...",
  "refreshToken": "eyJ...",
  "user": { "id": "uuid", "fullName": "Priya Sharma", "role": "buyer", "isVerified": true }
}
```

**Errors:**
- 400: Validation failed
- 401: Incorrect password
- 403: Account suspended
- 404: Account not found

---

### POST /api/auth/refresh
**Purpose:** Rotate access token using refresh token
**Authentication:** Public (refresh token in body)

**Response (200):** `{ "accessToken": "eyJ..." }`

**Errors:**
- 401: Refresh token invalid or expired — force re-login

---

### POST /api/auth/logout
**Purpose:** Invalidate refresh token
**Authentication:** Authenticated

**Side Effects:** Deletes refresh token from Redis

---

### GET /api/listings
**Purpose:** Browse + search listings catalog
**Authentication:** Public (guest browsing) or Authenticated

**Query Parameters:**
- `city` (string), `categoryL1`, `categoryL2`, `condition`, `minDaily`, `maxDaily`
- `size`, `rating` (min stars), `startDate`, `endDate` (availability check)
- `sort` (relevance|price_asc|rating|newest|distance), `page`, `limit` (default 20)
- `q` (search query, supports Hindi/English)

**Response (200):**
```json
{
  "listings": [
    {
      "id": "uuid",
      "title": "Bridal Lehenga — Red & Gold",
      "condition": "like_new",
      "itemValue": 25000,
      "dailyRatePct": 0.02,
      "dailyRateAmount": 500,
      "category": { "l1": "womens_fashion", "l2": "bridal_lehenga" },
      "seller": { "id": "uuid", "storeName": "Sunita Boutique", "avgRating": 4.7, "totalRentals": 23 },
      "photos": [{ "url": "https://cdn.rentkart.in/..." }],
      "isBoosted": false,
      "city": "Pune"
    }
  ],
  "total": 142,
  "page": 1,
  "limit": 20
}
```

---

### GET /api/listings/:id
**Purpose:** Get full listing detail + rental calculator data
**Authentication:** Public

**Response (200):** Full listing object including all photos, seller profile, reviews (paginated), availability blocks

---

### POST /api/listings
**Purpose:** Seller creates a new listing
**Authentication:** Authenticated (Seller)

**Request Body:**
```json
{
  "title": "Bridal Lehenga Red & Gold",
  "description": "Stunning bridal lehenga with...",
  "categoryL1": "womens_fashion",
  "categoryL2": "bridal_lehenga",
  "condition": "like_new",
  "itemValue": 25000,
  "dailyRatePct": 0.02,
  "locationPincode": "411001",
  "city": "Pune",
  "inventoryCount": 1
}
```

**Validation:**
- All required fields per schema
- If seller's published listing count < 3: `status = 'pending_review'`; else `status = 'live'`

**Response (201):** Created listing object with `status`

**Side Effects:**
- If pending_review: admin notification triggered
- Listing ID returned for photo upload

---

### POST /api/listings/:id/photos
**Purpose:** Upload photos to an existing listing
**Authentication:** Authenticated (Seller, must own listing)
**Content-Type:** multipart/form-data

**Validation:** Min 1 image per call; each file JPEG/PNG/WEBP; max 5MB per file; max 15 photos total per listing

**Response (200):** Array of uploaded photo objects with S3 URLs

---

### POST /api/orders
**Purpose:** Buyer places a rental order
**Authentication:** Authenticated (Buyer)

**Request Body:**
```json
{
  "listingId": "uuid",
  "startDate": "2024-11-15",
  "endDate": "2024-11-18",
  "deliveryAddress": {
    "line1": "Flat 4B, Orchid Heights",
    "line2": "Koregaon Park",
    "city": "Pune",
    "state": "Maharashtra",
    "pincode": "411001"
  }
}
```

**Validation:**
- Listing must be `live`, not blocked on requested dates
- startDate must be ≥ tomorrow
- Delivery pincode must be serviceable
- If item_value ≥ ₹10,000: Aadhaar verification must be completed

**Response (201):**
```json
{
  "orderId": "uuid",
  "orderNumber": "RK-2024-00423",
  "razorpayOrderId": "order_XXXX",
  "totalAmount": 25150,
  "breakdown": {
    "refundableHold": 25000,
    "deliveryCharge": 150
  }
}
```

**Side Effects:**
- Listing dates blocked in `listing_availability`
- Razorpay order created
- 15-minute hold placed; released if payment not captured

**Errors:**
- 400: Validation failed
- 409: Dates unavailable (race condition — dates just booked by another user)
- 402: ID verification required

---

### POST /api/orders/:id/payment-confirm
**Purpose:** Confirm payment captured (called after Razorpay success callback)
**Authentication:** Authenticated (Buyer)

**Request Body:**
```json
{
  "razorpayPaymentId": "pay_XXXX",
  "razorpaySignature": "hash"
}
```

**Side Effects:** Order status → `confirmed`; seller + buyer notifications sent; auto-sale date set

---

### POST /api/orders/:id/schedule-return
**Purpose:** Buyer schedules return pickup
**Authentication:** Authenticated (Buyer)

**Request Body:** `{ "returnDate": "2024-11-18" }`

**Validation:** returnDate must be ≤ auto_sale_date and ≥ today

**Response (200):** Updated order with `endDate` and recalculated estimated refund

---

### POST /api/orders/:id/inspections
**Purpose:** Delivery agent captures inspection photos
**Authentication:** Authenticated (Delivery Agent)
**Content-Type:** multipart/form-data

**Fields:** `inspectionType` (delivery|return), `damageLevel`, `damageNotes`, `qrTagScanned`, `photos[]` (min 8)

**Validation:**
- Must be assigned delivery agent for this order
- Photos must be captured in-app (EXIF metadata validated)
- QR tag check: if listing.qr_tag_id is set, scanned tag must match

**Side Effects:**
- Photos stored in S3
- If `inspectionType = 'return'`: 24-hour seller review window started
- If QR mismatch: admin alert + order flagged

---

### POST /api/orders/:id/disputes
**Purpose:** Buyer or seller raises a dispute
**Authentication:** Authenticated (Buyer or Seller)

**Request Body:**
```json
{
  "disputeType": "damage_dispute",
  "description": "The embroidery was intact when I returned it. I can see the delivery photo shows no damage.",
  "evidenceUrls": ["https://cdn.rentkart.in/..."]
}
```

**Validation:** Must be raised within 48 hours of inspection completion

**Side Effects:** Admin notification; refund/payout held pending resolution

---

### GET /api/seller/dashboard
**Purpose:** Seller dashboard summary
**Authentication:** Authenticated (Seller)

**Response (200):**
```json
{
  "earningsThisMonth": 12400,
  "totalRentals": 24,
  "pendingPayout": 1860,
  "avgRating": 4.8,
  "newOrders": 2,
  "activeRentals": 3
}
```

---

### PATCH /api/admin/listings/:id/approve
**Purpose:** Admin approves a listing pending review
**Authentication:** Authenticated (Admin)

**Response (200):** Updated listing with `status: 'live'`

**Side Effects:** Seller notification; listing live in catalog; `admin_logs` entry created

---

### POST /api/webhooks/razorpay
**Purpose:** Receive Razorpay payment/refund webhook events
**Authentication:** Webhook signature verification (HMAC-SHA256)

**Handled Events:**
- `payment.captured` → confirm order
- `payment.failed` → release listing dates; notify buyer
- `refund.processed` → update refund_status; notify buyer
- `payout.processed` → update payout_status; notify seller

---

## 4. Authentication & Authorization

### JWT Payload Structure
```json
// Access Token (15 min)
{
  "sub": "user-uuid",
  "role": "buyer",
  "isVerified": true,
  "iat": 1710000000,
  "exp": 1710000900
}

// Refresh Token (30 days) — stored in Redis with key: refresh:{userId}:{tokenId}
{
  "sub": "user-uuid",
  "tokenId": "uuid",
  "iat": 1710000000,
  "exp": 1712592000
}
```

### Authorization Levels
| Level | Middleware | Routes |
|-------|-----------|--------|
| Public | None | GET /listings, GET /listings/:id, POST /auth/* |
| Authenticated | `verifyAccessToken` | All buyer/seller actions |
| Seller Only | `verifyAccessToken` + `requireRole('seller')` | POST /listings, PATCH /listings/:id |
| Admin Only | `verifyAccessToken` + `requireRole('admin')` | /admin/* |
| Delivery Agent | `verifyAccessToken` + `requireRole('delivery')` | POST /orders/:id/inspections |

### Password Security
- bcrypt, 12 rounds
- Min 8 chars, must include uppercase, lowercase, number, special character
- Reset flow: OTP to phone → set new password → invalidate all refresh tokens

---

## 5. Data Validation Rules

| Field | Rule |
|---|---|
| Phone | `/^[6-9]\d{9}$/` (Indian mobile) |
| Password | Min 8 chars, regex `/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/` |
| Item Value | Integer, 500 ≤ value ≤ 500000 |
| Daily Rate % | Decimal, 0.005 ≤ rate ≤ 0.05 |
| Description | Min 100 characters |
| Title | Max 80 characters |
| Pincode | `/^\d{6}$/` |
| IFSC | `/^[A-Z]{4}0[A-Z0-9]{6}$/` |
| Date fields | ISO 8601 (YYYY-MM-DD) |
| Review rating | Integer 1–5 |

All inputs sanitised via Zod schemas before reaching business logic. SQL injection prevented by Prisma parameterised queries.

---

## 6. Error Handling

**Standard Error Response Format:**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": [
      { "field": "phone", "message": "Must be a valid 10-digit Indian mobile number" },
      { "field": "password", "message": "Must include at least one special character" }
    ]
  }
}
```

**Error Code → HTTP Status Mapping:**
| Code | HTTP Status |
|------|-------------|
| VALIDATION_ERROR | 400 |
| UNAUTHORIZED | 401 |
| FORBIDDEN | 403 |
| NOT_FOUND | 404 |
| CONFLICT | 409 |
| UNPROCESSABLE_ENTITY | 422 |
| RATE_LIMITED | 429 |
| INTERNAL_ERROR | 500 |
| SERVICE_UNAVAILABLE | 503 |

---

## 7. Caching Strategy

| Cache Key | Data | TTL | Invalidated When |
|-----------|------|-----|-----------------|
| `otp:{phone}` | 6-digit OTP | 60 seconds | Used or expired |
| `otp:attempts:{phone}` | Attempt count | 30 minutes | Reset on successful verify |
| `refresh:{userId}:{tokenId}` | Refresh token | 30 days | Logout, password change |
| `rate:login:{ip}` | Login attempt count | 15 minutes | TTL expiry |
| `listing:{id}` | Listing detail | 5 minutes | PATCH /listings/:id |
| `homepage:collections` | Curated homepage data | 5 minutes | Admin content update |
| `seller:dashboard:{sellerId}` | Dashboard metrics | 2 minutes | Order state change |

---

## 8. Rate Limiting

| Endpoint Group | Limit | Window | Per |
|---|---|---|---|
| POST /auth/login | 5 requests | 15 minutes | IP |
| POST /auth/register | 10 requests | 1 hour | IP |
| POST /auth/verify-otp (OTP send) | 3 requests | 10 minutes | Phone number |
| GET /api/* (public, unauthenticated) | 100 requests | 1 minute | IP |
| All authenticated API routes | 500 requests | 1 minute | User ID |
| POST /api/orders | 10 requests | 1 hour | User ID |
| POST /api/webhooks/razorpay | 1000 requests | 1 minute | IP (whitelisted Razorpay IPs only) |

---

## 9. Database Migrations

- **Tool:** Prisma Migrate
- **Process:** `prisma migrate dev` (local) → `prisma migrate deploy` (staging/prod via CI/CD pipeline)
- **Naming convention:** `YYYYMMDDHHMMSS_descriptive_name`
- **Rollback plan:** Each migration includes a rollback note in comments; destructive changes (DROP TABLE, DROP COLUMN) require admin approval and must be preceded by a backfill migration
- **Staging soak:** All migrations run in staging for minimum 24 hours before production

---

## 10. Backup & Recovery

- **Frequency:** RDS automated daily snapshots; transaction logs continuously streamed
- **Retention:** 30 days of daily snapshots; 7 days of transaction logs
- **Location:** AWS S3 in `ap-south-1`; cross-region copy to `ap-southeast-1` (Singapore)
- **Point-in-Time Recovery:** Enabled on RDS — can restore to any second within 30 days
- **Restore SLA:** Target RPO < 1 hour; RTO < 2 hours
- **Restore Procedure:** Documented in `/ops/DISASTER_RECOVERY.md`; tested quarterly
- **Escrow data:** Payments table backed up with double-confirmation; reconciled daily against Razorpay dashboard
