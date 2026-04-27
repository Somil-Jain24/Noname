# Implementation Plan & Build Sequence
# RentKart — India's Premier Rental Marketplace
**Version:** 1.0 | **MVP Target:** Week 12 from project kickoff | **Build Philosophy:** Backend-first, data-safe, test each step before proceeding

---

## Overview

RentKart handles escrow-secured financial transactions. This build sequence prioritises data integrity and financial correctness at every step. No feature ships until the data layer beneath it is confirmed correct. The UI is built to match established flows — never invented on the fly.

---

## Phase 1: Project Setup & Foundation
**Duration:** 1 week | **Goal:** All tooling configured, database running, teams unblocked

---

### Step 1.1: Repository & Monorepo Initialisation
**Duration:** 2 hours
**Goal:** Single repository with all workspaces scaffolded and running

**Tasks:**
1. Create GitHub organisation and repository (`rentkart-platform`)
2. Initialise monorepo structure:
   ```bash
   mkdir rentkart-platform && cd rentkart-platform
   git init
   mkdir apps/api apps/admin apps/mobile packages/shared-types
   ```
3. Add root `package.json` with workspaces:
   ```json
   { "workspaces": ["apps/*", "packages/*"] }
   ```
4. Initialise each app:
   ```bash
   # API
   cd apps/api && npm init -y
   # Admin
   npx create-next-app@14.1.4 apps/admin --typescript --tailwind --app
   # Mobile
   npx create-expo-app@50.0.14 apps/mobile --template expo-template-blank-typescript
   ```
5. Install ESLint 8.57.0, Prettier 3.2.5, Husky 9.0.11, lint-staged in root
6. Add `.eslintrc`, `.prettierrc`, `.gitignore`, `husky` pre-commit hook

**Success Criteria:**
- [ ] All three apps start without errors
- [ ] `npm run lint` passes across all workspaces
- [ ] Git pre-commit hook runs linting on staged files
- [ ] `develop` and `main` branches created; branch protection rules set on `main`

**Reference:** TECH_STACK.md sections 2, 5

---

### Step 1.2: Environment & Secrets Setup
**Duration:** 1 hour
**Goal:** All environment variables configured; no secrets committed to Git

**Tasks:**
1. Create `.env.example` in `apps/api/` with all variables from TECH_STACK.md section 6
2. Add `.env` to `.gitignore` in all workspaces
3. Set up GitHub Secrets for CI/CD:
   - `DATABASE_URL`, `REDIS_URL`, `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`
   - `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `JWT_ACCESS_SECRET`, `JWT_REFRESH_SECRET`
4. Provision AWS S3 bucket `rentkart-media-dev` in `ap-south-1` with:
   - Private ACL (all objects private)
   - CORS policy allowing API origin
   - CloudFront distribution pointed at bucket
5. Provision Razorpay test mode credentials (not production)

**Success Criteria:**
- [ ] `.env.example` documents every variable (no value, just keys)
- [ ] No real secrets in any committed file (GitHub secret scan enabled)
- [ ] S3 bucket accessible from dev machine with test upload

**Reference:** TECH_STACK.md section 6

---

### Step 1.3: Database Setup & Schema
**Duration:** 4 hours
**Goal:** PostgreSQL running; all tables created via Prisma; visible in Prisma Studio

**Tasks:**
1. Provision PostgreSQL 16.2 locally (Docker):
   ```bash
   docker run --name rentkart-db -e POSTGRES_PASSWORD=localpass -e POSTGRES_DB=rentkart_dev -p 5432:5432 -d postgres:16.2
   ```
2. Install Prisma 5.11.0:
   ```bash
   cd apps/api && npm install prisma@5.11.0 @prisma/client@5.11.0
   npx prisma init
   ```
3. Write `schema.prisma` implementing all tables from BACKEND_STRUCTURE.md section 2:
   - users, kyc_documents, seller_profiles, listings, listing_photos, listing_availability
   - orders, order_inspections, disputes, reviews, payments, notifications, admin_logs
4. Run initial migration:
   ```bash
   npx prisma migrate dev --name init_schema
   ```
5. Write `prisma/seed.ts` with:
   - 1 admin user, 2 test sellers (KYC approved), 3 test buyers
   - 5 sample listings (status: live) with photos
6. Run seed: `npm run db:seed`
7. Verify in Prisma Studio: `npm run db:studio`

**Success Criteria:**
- [ ] All 13 tables created with correct columns and constraints
- [ ] Foreign key relationships verified
- [ ] Seed data visible in Prisma Studio
- [ ] Migration file in `prisma/migrations/` committed to repo
- [ ] `npm run db:migrate` script documented in package.json

**Reference:** BACKEND_STRUCTURE.md section 2

---

### Step 1.4: CI/CD Pipeline Setup
**Duration:** 2 hours
**Goal:** GitHub Actions pipeline running; PRs blocked until lint + tests pass

**Tasks:**
1. Create `.github/workflows/ci.yml`:
   ```yaml
   on: [push, pull_request]
   jobs:
     ci:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v4
         - uses: actions/setup-node@v4
           with: { node-version: '20.11.1' }
         - run: npm ci
         - run: npm run lint
         - run: npm run type-check
         - run: npm run test
   ```
2. Create `.github/workflows/deploy-staging.yml` (trigger: push to `develop`)
3. Configure branch protection on `main`: require 2 approvals + CI pass
4. Configure branch protection on `develop`: require 1 approval + CI pass

**Success Criteria:**
- [ ] CI pipeline runs and passes on a test PR
- [ ] PRs to `develop` blocked without approval + CI green
- [ ] `main` branch additionally requires 2 approvals

---

## Phase 2: Backend Foundation — Auth & Core APIs
**Duration:** 1.5 weeks | **Goal:** Authentication working end-to-end; core CRUD APIs functional

---

### Step 2.1: Fastify Server Setup
**Duration:** 3 hours
**Goal:** API server running with health check, CORS, rate limiting, and logging

**Tasks:**
1. Install dependencies (exact versions from TECH_STACK.md section 8):
   ```bash
   npm install fastify@4.26.2 @fastify/cors@9.0.1 @fastify/rate-limit@9.1.0 @fastify/jwt@8.0.1 @fastify/multipart@8.2.0 zod@3.22.4 winston@3.13.0 ioredis@5.3.2
   ```
2. Create `src/server.ts` with Fastify instance, CORS config, rate limiting
3. Add `GET /health` endpoint returning `{ status: 'ok', timestamp: ISO_STRING }`
4. Configure Winston JSON logging (request/response logs)
5. Connect Redis client via ioredis

**Success Criteria:**
- [ ] `npm run dev` starts server on port 3000
- [ ] `GET /health` returns 200
- [ ] Request logs appear in JSON format
- [ ] Redis connection confirmed in startup log

**Reference:** TECH_STACK.md sections 3, 9

---

### Step 2.2: Authentication Endpoints
**Duration:** 1 day
**Goal:** Register, OTP verify, login, refresh, logout all working; tested with Bruno

**Tasks:**
1. Install auth dependencies: `bcrypt@5.1.1 jsonwebtoken@9.0.2`
2. Implement Zod schemas for all auth request bodies
3. Implement endpoints per BACKEND_STRUCTURE.md section 3:
   - `POST /api/auth/register` — create user, send OTP (mock SMS in dev), store OTP in Redis (60s TTL)
   - `POST /api/auth/verify-otp` — verify OTP, return JWT pair
   - `POST /api/auth/login` — phone + password, return JWT pair
   - `POST /api/auth/refresh` — rotate access token
   - `POST /api/auth/logout` — delete refresh token from Redis
4. Implement `verifyAccessToken` Fastify middleware (hook)
5. Implement `requireRole(role)` middleware
6. Write Vitest unit tests for: OTP generation, JWT creation, bcrypt verify

**Success Criteria:**
- [ ] Can register a new buyer account
- [ ] OTP stored in Redis with 60s TTL (verify in Redis CLI)
- [ ] Login returns valid JWT pair
- [ ] Protected route returns 401 without token
- [ ] Role middleware returns 403 for wrong role
- [ ] Unit tests pass for all auth utilities

**Reference:** BACKEND_STRUCTURE.md sections 3, 4

---

### Step 2.3: Listings APIs
**Duration:** 1.5 days
**Goal:** Sellers can create listings; buyers can browse and view PDP data

**Tasks:**
1. Implement endpoints per BACKEND_STRUCTURE.md section 3:
   - `GET /api/listings` — paginated browse with all filters + full-text search
   - `GET /api/listings/:id` — full listing detail
   - `POST /api/listings` — seller creates listing (Zod validation)
   - `PATCH /api/listings/:id` — seller edits listing
   - `POST /api/listings/:id/photos` — S3 upload via multipart
2. Implement PostgreSQL full-text search with `tsvector` for title + description
3. Add Redis cache for `listing:{id}` (5-min TTL); invalidate on update
4. Implement listing status workflow: `draft → pending_review → live`
5. Write Vitest integration tests for listing creation and browse

**Success Criteria:**
- [ ] Seller can create listing; status = `pending_review` if seller has < 3 listings
- [ ] Browse returns paginated results with filters applied
- [ ] Search returns fuzzy-matched results
- [ ] Photo upload stores file in S3 and returns CDN URL
- [ ] Redis cache hit confirmed for repeated listing detail requests

**Reference:** BACKEND_STRUCTURE.md section 3; PRD.md Feature 2

---

### Step 2.4: Order & Checkout APIs
**Duration:** 2 days
**Goal:** Buyers can place orders; Razorpay orders created; payment confirmation working

**Tasks:**
1. Install `razorpay@2.9.2`
2. Implement `POST /api/orders` (per BACKEND_STRUCTURE.md):
   - Validate listing availability for requested dates (atomic lock)
   - Calculate: item_value, delivery_charge, auto_sale_date
   - Create Razorpay order
   - Block listing dates in `listing_availability`
   - Return Razorpay order ID to client
3. Implement `POST /api/orders/:id/payment-confirm` — verify Razorpay signature; update order status to `confirmed`
4. Implement Razorpay webhook handler `POST /api/webhooks/razorpay`:
   - `payment.captured`, `payment.failed`
5. Implement availability race condition guard (Postgres row-level lock)
6. Write integration tests for checkout flow; mock Razorpay SDK in tests

**Success Criteria:**
- [ ] Order created with correct escrow amount
- [ ] Listing dates blocked immediately on order placement
- [ ] Razorpay order ID returned to client
- [ ] Payment confirmation updates order status to `confirmed`
- [ ] Payment failure releases blocked dates
- [ ] Webhook signature verification tested with test payload

**Reference:** BACKEND_STRUCTURE.md sections 3, 2 (orders table); PRD.md Feature 1

---

### Step 2.5: Inspection, Dispute & Refund APIs
**Duration:** 1.5 days
**Goal:** Delivery agents can submit inspections; disputes can be raised; refund calculation correct

**Tasks:**
1. `POST /api/orders/:id/schedule-return` — buyer sets return date
2. `POST /api/orders/:id/inspections` — agent uploads photos (S3), scans QR tag, sets damage level
3. Implement seller 24-hour review window logic
4. Implement refund calculation function: `item_value - usage_charge - damage_penalty - late_fee`
5. Implement auto-sale scheduler (daily cron job): check orders where `auto_sale_date ≤ today` and `auto_sale_triggered = false`
6. `POST /api/orders/:id/disputes` — create dispute, hold refund
7. `PATCH /api/admin/disputes/:id/resolve` — admin resolves dispute, triggers adjusted refund/payout
8. Write unit tests for refund calculation formula with multiple scenarios (no damage, damage, late return, auto-sale)

**Success Criteria:**
- [ ] Refund calculation matches formula in PRD.md section 8
- [ ] Auto-sale cron correctly identifies Day 25+ orders
- [ ] Inspection photos land in S3 with GPS + timestamp metadata
- [ ] QR tag mismatch triggers admin notification
- [ ] Dispute raised → refund held → admin resolves → payout adjusted

**Reference:** BACKEND_STRUCTURE.md sections 3, 5; PRD.md Feature 7

---

## Phase 3: Admin Panel (Web)
**Duration:** 1 week | **Goal:** Admins can handle all operational tasks without DB access

---

### Step 3.1: Admin Panel Setup & Auth
**Duration:** 4 hours
**Goal:** Next.js admin app running; admin login working; protected routes enforced

**Tasks:**
1. Set up Next.js 14.1.4 admin app (already scaffolded in Step 1.1)
2. Install: `shadcn/ui@0.8.0`, `@tanstack/react-query@5.28.6`, `recharts@2.12.2`
3. Implement admin login page (phone + password)
4. Add Axios instance with JWT interceptor (auto-refresh on 401)
5. Implement route protection middleware (Next.js `middleware.ts`)
6. Create base layout: sidebar navigation + main content area

**Success Criteria:**
- [ ] Admin login works with seeded admin credentials
- [ ] Non-admin routes redirect to login
- [ ] Sidebar shows all navigation items

---

### Step 3.2: KYC & Listing Review Queue
**Duration:** 1 day
**Goal:** Admins can review and approve/reject KYC submissions and new listings

**Tasks:**
1. Build KYC review page: list of pending KYC docs with preview + Approve/Reject actions
2. Build listing review queue: pending listings with full detail view + Approve/Reject
3. Connect to `PATCH /api/admin/kyc/:id` and `PATCH /api/admin/listings/:id/approve`
4. Add optimistic updates via React Query mutations

**Success Criteria:**
- [ ] Admin sees pending KYC in real-time
- [ ] Approve → seller immediately gets KYC approved notification
- [ ] Reject with reason → seller notified with reason shown in app

---

### Step 3.3: Order Management & Dispute Resolution
**Duration:** 1.5 days
**Goal:** Admins can view all orders, intervene, and resolve disputes

**Tasks:**
1. Build orders dashboard: status filter, search by Order ID, buyer, seller
2. Build order detail: full timeline view, delivery vs return photo side-by-side viewer
3. Build dispute workspace: evidence viewer, resolution actions (refund / penalty / no action)
4. Connect all admin actions to backend endpoints
5. Build `admin_logs` audit trail viewer

**Success Criteria:**
- [ ] Admin can view any order's full photo evidence
- [ ] Resolve dispute → buyer refund and seller payout recalculated and shown
- [ ] All admin actions appear in audit log

---

### Step 3.4: Analytics Dashboard
**Duration:** 4 hours
**Goal:** Admin can view GMV, commissions, daily orders, and seller health

**Tasks:**
1. Build analytics page with Recharts:
   - Daily GMV (line chart, last 30 days)
   - Order volume by status (bar chart)
   - Commission earned (running total)
   - Top sellers by GMV
2. Connect to `GET /api/admin/analytics` endpoint
3. Add date range picker

**Success Criteria:**
- [ ] All charts load with real data from seeded orders
- [ ] Date range filter updates charts

**Reference:** PRD.md Feature 8

---

## Phase 4: Mobile Apps (Buyer & Seller)
**Duration:** 2.5 weeks | **Goal:** Buyer and seller apps fully functional end-to-end

---

### Step 4.1: Design System in React Native
**Duration:** 1 day
**Goal:** NativeWind configured; all core components from FRONTEND_GUIDELINES.md built

**Tasks:**
1. Install NativeWind 4.0.1 and configure Tailwind for React Native
2. Build component library (each component with all variants + states):
   - Button (Primary, Secondary, Outline, Ghost, Danger — all sizes, loading state)
   - Input (text, password, textarea — error, success, disabled states)
   - Card (listing card, info card, escrow breakdown card)
   - Toast/Alert (success, warning, error, info)
   - Loading skeletons (listing card skeleton, page skeleton)
   - Empty state component
3. Verify all components against FRONTEND_GUIDELINES.md section 4

**Success Criteria:**
- [ ] All components render correctly on iOS and Android simulator
- [ ] Color tokens match FRONTEND_GUIDELINES.md section 2 exactly
- [ ] All states (loading, error, empty) implemented for every component
- [ ] Touch targets are minimum 44×44px on all interactive elements

**Reference:** FRONTEND_GUIDELINES.md sections 2, 3, 4

---

### Step 4.2: Buyer App — Auth & Onboarding
**Duration:** 1 day
**Goal:** Buyer registration, OTP verification, and login working end-to-end

**Tasks:**
1. Build screens: Splash, Role Selection, Phone Entry, OTP Verification, Password Setup, Profile Setup
2. Connect to auth API endpoints
3. Implement deep link handling for OTP (SMS auto-read via Android)
4. Store JWT tokens in `expo-secure-store` (not AsyncStorage)
5. Set up React Navigation bottom tab bar for buyer

**Success Criteria:**
- [ ] Full registration flow completes end-to-end with real API
- [ ] OTP verification succeeds; tokens stored securely
- [ ] Login flow works; redirects to home
- [ ] Navigation structure matches APP_FLOW.md section 3

**Reference:** APP_FLOW.md Flow 1; FRONTEND_GUIDELINES.md section 8

---

### Step 4.3: Buyer App — Browse, Search & PDP
**Duration:** 2 days
**Goal:** Buyer can discover listings, search, filter, and view full PDP with rental calculator

**Tasks:**
1. Build Homepage with curated sections (API-driven); skeleton loading states
2. Build Category Browse (PLP) with filter bottom sheet and sort options
3. Build Search screen with debounced input (300ms), results grid
4. Build Product Detail Page (PDP):
   - Photo carousel (swipe + pinch-to-zoom)
   - Seller info card
   - Rental Calculator (date picker → live calculation)
   - Availability calendar (blocked dates greyed out)
   - Reviews section (paginated)
   - "Rent Now" sticky bottom CTA
5. Implement image caching with `expo-image`

**Success Criteria:**
- [ ] Search returns results within 500ms (debounced)
- [ ] Rental calculator recalculates instantly on date change
- [ ] Blocked dates unselectable on availability calendar
- [ ] PDP photos load progressively; no janky scroll

**Reference:** APP_FLOW.md Flows 2, 3; PRD.md Feature 2

---

### Step 4.4: Buyer App — Checkout & Payment
**Duration:** 1.5 days
**Goal:** Buyer completes checkout and payment end-to-end; Razorpay SDK integrated

**Tasks:**
1. Build Address Selection screen (list saved addresses + "Add New" form)
2. Build Order Summary / Checkout screen with escrow breakdown card
3. Integrate Razorpay React Native SDK for payment
4. Implement payment success → `POST /api/orders/:id/payment-confirm` → Order Confirmation screen
5. Handle payment failure and timeout states
6. Implement DigiLocker OTP flow for items ≥₹10,000

**Success Criteria:**
- [ ] End-to-end checkout completes with Razorpay test mode
- [ ] Order Confirmation screen shows all required info
- [ ] Payment failure shows error with retry option
- [ ] ID verification screen appears for ≥₹10,000 item

**Reference:** APP_FLOW.md Flow 4; PRD.md Feature 1

---

### Step 4.5: Buyer App — Orders, Returns & Notifications
**Duration:** 1.5 days
**Goal:** Buyers can track orders, schedule returns, and receive push notifications

**Tasks:**
1. Build My Orders screen (active + past; status-filtered tabs)
2. Build Order Detail screen (full timeline, countdown timer, "Schedule Return" CTA)
3. Build Return Scheduling screen (date picker + estimated refund preview)
4. Build Dispute Raise screen
5. Implement FCM push notifications (Day 20/23/24 reminders, confirmation, refund)
6. Implement in-app chat proxy (seller contact masked)

**Success Criteria:**
- [ ] Day 25 countdown visible on active order detail
- [ ] Return scheduling updates estimated refund in real time
- [ ] Push notification received on device for order confirmation
- [ ] In-app chat sends and receives messages (no phone numbers exposed)

**Reference:** APP_FLOW.md Flows 5, 8; PRD.md Feature 3

---

### Step 4.6: Seller App
**Duration:** 2 days
**Goal:** Seller can onboard, create listings, manage orders, and view earnings

**Tasks:**
1. Build Seller Registration + KYC Upload + Bank Details flows
2. Build Seller Dashboard home (metrics summary + new order alerts)
3. Build Add Listing wizard (multi-step: category → photos → pricing → availability → review)
4. Build in-app listing photo upload (camera capture only; guided angles)
5. Build Seller Orders screen (new orders with Accept/Decline; active rentals)
6. Build Order Detail for seller (return photos review within 24-hour window)
7. Build Earnings / Payout history screen

**Success Criteria:**
- [ ] Full seller onboarding completes including bank verification
- [ ] New listing created and awaiting admin review
- [ ] Seller can accept/decline order within 2-hour window
- [ ] Return photos visible in 24-hour review window
- [ ] Earnings dashboard shows correct amounts matching backend

**Reference:** APP_FLOW.md Flows 6, 7; PRD.md Feature 5

---

### Step 4.7: Delivery Agent PWA
**Duration:** 1 day
**Goal:** Delivery agent can see assignments, complete delivery + return inspections

**Tasks:**
1. Build lightweight React PWA (separate from main apps)
2. Today's Assignments list screen
3. Delivery Inspection flow: guided 8-photo capture (in-app camera only), GPS tag, QR scan
4. Buyer OTP confirmation screen
5. Return Inspection flow: same photo guide + damage classification + notes

**Success Criteria:**
- [ ] Agent can complete delivery inspection end-to-end
- [ ] Photos upload to S3 with GPS + timestamp metadata
- [ ] QR tag mismatch triggers error and blocks completion
- [ ] Return photos visible to seller within minutes of upload

**Reference:** APP_FLOW.md section 3 (Delivery Agent App); PRD.md Feature 4

---

## Phase 5: Testing
**Duration:** 1 week

---

### Step 5.1: Unit Tests
**Duration:** 3 days
**Goal:** 80% coverage on business-critical modules

**Targets:**
| Module | Coverage Target | Key Tests |
|---|---|---|
| Auth utilities | 95% | OTP generate/verify, JWT sign/verify, bcrypt |
| Refund calculator | 100% | All formula variants: no damage, damage, late fee, auto-sale |
| Listing search | 90% | Fuzzy match, filter combinations, pagination |
| Razorpay webhook | 95% | Signature verify, each event type |
| Order state machine | 90% | Valid and invalid transitions |
| Rate limiter | 85% | Hit limit, reset, per-user vs per-IP |

Run: `npm run test:coverage` — must reach 80% overall before deploy

---

### Step 5.2: Integration Tests (API)
**Duration:** 2 days
**Goal:** Each core flow tested end-to-end against a real test DB

**Flows to test:**
1. Buyer registers → verifies OTP → logs in → browses listings → places order → confirms payment
2. Seller creates listing → admin approves → order received → order accepted → payout processed
3. Order placed → item returned → inspection photos uploaded → refund calculated and triggered
4. Dispute raised → admin resolves → adjusted refund processed
5. Day 25 auto-sale cron triggers → order converted → seller payout released

Each test uses a clean test DB seeded fresh before the suite runs.

---

### Step 5.3: Manual QA Checklist
**Duration:** 2 days
**Goal:** Human QA on real devices confirms all flows match APP_FLOW.md exactly

**Device targets:** Android (Pixel 6, Samsung mid-range), iOS (iPhone 12+), low-end Android (for performance baseline)

**QA sign-off required for:** checkout with Razorpay test mode, photo inspection flow, admin dispute resolution, FCM notification delivery on Android + iOS

---

## Phase 6: Deployment

---

### Step 6.1: Staging Environment
**Duration:** 2 days
**Goal:** Full stack running in AWS staging; smoke tested

**Tasks:**
1. Provision AWS EC2 `t3.medium` + RDS PostgreSQL 16.2 + ElastiCache Redis in `ap-south-1`
2. Create Docker image: `apps/api` → push to AWS ECR
3. Deploy API container to EC2; configure Nginx reverse proxy with TLS (SSL cert via Let's Encrypt)
4. Deploy admin Next.js app to Vercel (staging branch)
5. Run `prisma migrate deploy` against staging DB
6. Run seed with realistic data
7. Deploy Expo app to internal testing track (EAS Build)
8. Smoke test: complete checkout → return → refund flow in staging

**Smoke Test Checklist:**
- [ ] Health endpoint returns 200
- [ ] New buyer can register and place an order in Razorpay test mode
- [ ] Admin can approve listing
- [ ] Delivery agent app loads and completes inspection
- [ ] FCM push received on test device

---

### Step 6.2: Production Launch
**Duration:** 1 day
**Goal:** Platform live with 25 sellers onboarded; buyer-facing launch

**Tasks:**
1. Switch Razorpay to live mode credentials
2. Run `prisma migrate deploy` on production DB
3. Deploy production Docker image with production env vars
4. Deploy admin panel to Vercel production
5. Submit mobile apps to Google Play and App Store (review time: 3–7 days — initiate before cutover)
6. Configure Sentry error alerts to Slack #ops channel
7. Enable UptimeRobot monitoring with PagerDuty integration
8. Run final smoke test on production with ₹1 test transaction

**Post-Launch Monitoring (First 48 Hours):**
- Monitor Sentry for new errors hourly
- Monitor RDS CPU and connection count
- Monitor Razorpay dashboard for webhook delivery
- Keep rollback Docker image tagged and ready

---

## Milestones

| Milestone | Target | Deliverables |
|-----------|--------|--------------|
| Foundation Complete | End of Week 1 | Repo, DB schema, CI/CD, env configured |
| Auth & Core APIs | End of Week 2.5 | All backend endpoints tested; admin panel skeleton |
| Admin Panel | End of Week 3.5 | Full admin ops possible without DB access |
| Mobile Apps Alpha | End of Week 6 | Buyer + Seller apps functional on simulators |
| Mobile Apps Beta | End of Week 8 | End-to-end flows on real devices; seller onboarding starts |
| Seller Onboarding | End of Week 10 | 25 verified sellers onboarded; 150+ listings live |
| Testing Complete | End of Week 11 | 80% test coverage; QA signed off |
| **MVP Launch** | **End of Week 12** | Live on App Store + Play Store; buyer-facing marketing begins |
| V1.1 | Month 5 | Referral program, Trustworthiness Score, subscription plan |

---

## Risk Mitigation

| Risk | Impact | Likelihood | Mitigation |
|------|--------|-----------|------------|
| Razorpay escrow compliance questioned | Critical | Medium | Confirm PA licence coverage with Razorpay account manager pre-launch; retain fintech lawyer |
| Apple App Store review rejects app | High | Medium | Submit 2 weeks before launch date; have PWA fallback ready |
| Cold start — sellers don't list | Critical | High | Founding Seller program launched in Week 8 (parallel to mobile dev); personal onboarding |
| Photo inspection quality insufficient | High | Medium | 8-angle photo guide built into agent PWA; tested in Step 5.3 on real deliveries |
| Razorpay webhook delivery failures | High | Low | Implement retry queue + daily reconciliation against Razorpay dashboard |
| DB migration failure in production | Critical | Low | Staging soak of 24h; rollback script prepared per migration; point-in-time recovery enabled |

---

## MVP Success Criteria

- [ ] All P0 features from PRD.md implemented and QA-signed off
- [ ] All core flows from APP_FLOW.md working end-to-end on real devices
- [ ] UI matches FRONTEND_GUIDELINES.md (design review sign-off)
- [ ] API matches BACKEND_STRUCTURE.md (no undocumented endpoints in production)
- [ ] 80% test coverage across business-critical modules
- [ ] Zero P0/P1 bugs open at launch
- [ ] Page load time < 2 seconds on 4G mobile
- [ ] Checkout → payment < 1 second API response
- [ ] 25 verified sellers live with 150+ listings
- [ ] Razorpay escrow compliance confirmed in writing
- [ ] Sentry monitoring active; UptimeRobot alerts configured
- [ ] Rollback plan documented and tested in staging
