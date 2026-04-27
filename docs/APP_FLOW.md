# Application Flow Documentation
# RentKart — India's Premier Rental Marketplace
**Version:** 1.0 | **Last Updated:** MVP Build Phase

---

## 1. Entry Points

| Entry Method | What User Sees |
|---|---|
| Direct URL / App Store Install | Onboarding splash → Role selection (Buyer / Seller) → Registration |
| Deep Link — Listing URL | Product Detail Page (PDP) for the specific item; login wall if not authenticated |
| Deep Link — Order Notification | Redirects to Order Detail Page; login required |
| Day 20/23/24 Return Reminder SMS/Push | Opens app to Order Detail Page with return countdown and "Schedule Return" CTA |
| Referral Link | Registration page with referral code pre-filled |
| Google/Facebook Ad | Category Browse Page or specific listing |

---

## 2. Core User Flows

---

### Flow 1: Buyer Registration & Onboarding

**Goal:** New user creates a buyer account and verifies identity
**Entry Point:** App install or referral link

#### Happy Path
1. **Splash Screen** — "Rent India's Best Fashion" headline, "Get Started" CTA → tap
2. **Role Selection** — "I want to Rent Items" / "I want to List Items" — tap Buyer
3. **Phone Number Entry** — Enter 10-digit mobile number → tap "Send OTP"
4. **OTP Verification** — 6-digit OTP field, 60-second resend timer → enter OTP → verified
5. **Password Setup** — Set password (8+ chars, uppercase, number, symbol) → confirm
6. **Profile Basics** — Full name, city, profile photo (optional) → "Continue"
7. **Homepage** — Arrives at buyer homepage, onboarding tooltips on first visit

#### Error States
- Invalid phone number → inline error "Enter a valid 10-digit Indian mobile number"
- OTP expired → show "Resend OTP" button with 30-second cooldown
- OTP incorrect → "Incorrect OTP. X attempts remaining" (max 5 before 30-min lockout)
- Weak password → inline checklist showing failed requirements
- Network offline → toast "No internet connection. Please check your connection."

#### Edge Cases
- User exits mid-flow → session preserved; resume from same step on re-open
- Phone number already registered → redirect to "Login instead" screen
- Referral code invalid → proceed without referral, show "Invalid code" toast (non-blocking)

#### Exit Points
- Success → Buyer Homepage
- Abandonment → Splash screen on next open with "Continue Registration" prompt

---

### Flow 2: Buyer — Browse & Product Discovery

**Goal:** Buyer finds a suitable item to rent
**Entry Point:** Homepage after login

#### Happy Path
1. **Homepage** — Curated sections: "Wedding Season Picks", "Trending This Week", "Budget Under ₹1500", "Top Rated Sellers"
2. **Category Browse** — Tap category card (e.g., "Bridal Lehenga") → Category Results Page
3. **Search** — Tap search bar → type keyword (Hindi/English) → results appear with fuzzy match
4. **Filter** — Tap filter icon → select: City, Size, Price range (daily ₹), Condition, Rating, Available dates → "Apply Filters"
5. **Sort** — Sort by: Relevance / Price Low–High / Rating / Newest / Distance
6. **Product Listing Card** — Shows: item photo, seller name + rating, daily rate, item value (escrow), availability indicator
7. **Tap Listing → Product Detail Page (PDP)**

#### Error States
- Search returns 0 results → empty state: "No items found for '[query]'" + "Try removing filters" CTA
- Images fail to load → placeholder skeleton + retry icon
- Filter returns 0 results → "No listings match these filters" + "Clear Filters" CTA

#### Edge Cases
- User searches in Hindi → transliterate to canonical English search term, show results
- Location permissions denied → use manually entered city from profile instead

---

### Flow 3: Buyer — Product Detail Page (PDP) & Rental Calculator

**Goal:** Buyer evaluates item and estimates rental cost
**Entry Point:** Listing card tap from browse/search

#### Happy Path
1. **Photo Carousel** — Swipe through 4–15 photos; pinch-to-zoom; optional video
2. **Item Details** — Title, condition, size/fit, occasion tags, care instructions, brand (if set)
3. **Seller Info Card** — Seller name, verified badge, rating, total rentals completed
4. **Rental Calculator** — Select Start Date + End Date via calendar picker:
   - System calculates: Rental Days, Daily Cost (item value × daily %), Total Usage Charge, Expected Refund
   - "Rental Fee: ₹X │ Refundable Hold: ₹Y │ You get ₹Y back"
5. **Availability Calendar** — Greyed-out dates = blocked; green = available
6. **Reviews Section** — Star rating breakdown, individual reviews with photos, "Verified Rental" badge
7. **"Rent Now" CTA** → triggers checkout flow

#### Error States
- Selected dates overlap with blocked dates → inline error, re-prompt date selection
- Seller inactive/listing archived → "This item is currently unavailable" with suggested alternatives
- Item value loading failure → show error state with retry

---

### Flow 4: Buyer — Checkout & Payment

**Goal:** Buyer places a confirmed rental order with payment
**Entry Point:** "Rent Now" on PDP

#### Happy Path
1. **Address Selection** — Choose saved address or "Add New Address" → enter address + pincode
2. **Delivery Feasibility Check** — System checks if pincode is serviceable → shows expected delivery date
3. **Order Summary** — Clear breakdown:
   - Escrow Hold (Refundable): ₹X
   - Delivery Charge: ₹Y
   - Estimated Usage Charge (calculated): ₹Z (shown as reference, deducted at refund)
   - **Total to Pay: ₹X + ₹Y**
   - "You will get ₹X back after return"
4. **25-Day Notice** — Prominent callout: "If not returned by [Date+25], item is considered purchased. No refund."
5. **ID Verification (if item value ≥ ₹10,000)** — Aadhaar OTP verification via DigiLocker
6. **Payment Method** — Select: UPI / Credit Card / Debit Card / Net Banking / Wallet
7. **Pay** → redirect to payment gateway → payment processed
8. **Order Confirmation Screen** — Order ID, expected delivery date, seller name, rental policy summary
9. **Confirmation push + SMS + email** sent

#### Error States
- Pincode not serviceable → "We don't deliver to this pincode yet" + option to try different address
- ID verification fails → "Verification failed. Please retry or contact support." (order blocked until verified)
- Payment fails → return to payment selection with error reason; allow retry
- Payment gateway timeout → show "We're checking your payment status" spinner → confirm or refund within 5 min
- Item booked by someone else after checkout started → "This item was just rented by another user. Sorry!" + redirect to catalog

#### Edge Cases
- User starts checkout but doesn't complete — order reserved for 15 minutes then released
- Multiple simultaneous checkouts for same item — first completed payment wins; others get error and refund if charged

---

### Flow 5: Buyer — In-Use & Return Flow

**Goal:** Buyer uses item and schedules return before Day 25
**Entry Point:** Order detail page during active rental

#### Happy Path
1. **Order Detail Page (Active)** — Shows: rental status, days elapsed, days remaining, current usage charge accrued, Day 25 countdown
2. **"Schedule Return"** → Calendar picker → select return date
3. **System confirms** return date + shows updated usage charge + expected refund amount
4. **Return Day** — Delivery agent arrives at scheduled time → opens inspection app
5. **Agent Captures Return Photos** — 8 photos from fixed angles, GPS + timestamp locked; QR tag scanned for item match
6. **Buyer Acknowledges** — Sees agent's damage classification on screen; confirms with in-app OTP
7. **Order marked "Returned"** → 24-hour seller review window begins
8. **Refund Triggered** → automated calculation; buyer sees final breakdown in app

#### Error States
- Buyer unavailable for return pickup → agent marks "Missed Pickup"; rescheduled within 24 hours; no extra charge for first miss
- Buyer refuses to acknowledge condition recording → agent escalates to admin; order held
- QR tag mismatch → agent cannot complete return; admin alerted; buyer questioned

#### Edge Cases
- Day 24 and no return scheduled → automated push + SMS escalation; "Schedule Return Now" deeplink
- Day 25 arrives with no return → order auto-converts to "Sold"; push + SMS notification to buyer and seller
- Item damaged during delivery (before use) → buyer raises dispute immediately at delivery; delivery agent confirms in app

---

### Flow 6: Seller — Onboarding

**Goal:** Seller creates a verified account and posts first listing
**Entry Point:** App install → "I want to List Items" or seller invite link

#### Happy Path
1. **Role Selection** → "I want to List Items"
2. **Phone OTP** → same as buyer registration
3. **Password Setup**
4. **KYC Upload:**
   - Aadhaar card (front + back photo)
   - PAN card photo
   - Address proof (utility bill or Aadhaar address)
   - Admin review within 1 business day → status: Pending → Verified / Rejected
5. **Bank Details** → Account number + IFSC → penny drop verification (₹1 test transfer)
6. **Store Profile** → Store name, city, bio, logo/photo, delivery preference (self / platform)
7. **First Listing** → guided listing creation (see Flow 7)
8. **Admin approves first listing** → seller is live

#### Error States
- KYC rejected → notification with reason + option to re-upload
- Penny drop fails → "Bank verification failed. Check account number and IFSC." → retry
- Photo quality too low → "Photo unclear. Please retake in good lighting."

---

### Flow 7: Seller — Create Listing

**Goal:** Seller publishes a new item for rent
**Entry Point:** Seller dashboard → "Add New Listing"

#### Happy Path
1. **Category Selection** → L1 (Women's Fashion / Men's Fashion / Accessories) → L2 (Bridal Lehenga, etc.)
2. **Photos** → minimum 4 required (front, back, detail, flat-lay); max 15; optional video
3. **Item Details** → Title, Description (≥100 chars), Condition, Size/fit
4. **Pricing** → Item Value in ₹ → Daily Rental % (slider 0.5%–5%; default 2%) → system shows sample usage calculation
5. **Availability Calendar** → Set available date ranges; optionally add blackout dates
6. **Location** → Pincode for delivery feasibility
7. **Optional Fields** → Brand, occasion tags, care instructions, bundle configuration
8. **Review & Publish** → Preview card + preview PDP → "Submit for Review" (first 3 listings) or "Publish Now"
9. **Admin review (first 3)** → approved → listing live; else → notification with reason

#### Error States
- Fewer than 4 photos → "Please upload at least 4 photos to continue"
- Item value outside valid range → "Item value must be between ₹500 and ₹5,00,000"
- Description too short → "Description must be at least 100 characters (currently X)"

---

### Flow 8: Dispute Resolution

**Goal:** Buyer or seller raises and resolves a damage/refund dispute
**Entry Point:** Order detail page → "Raise Dispute"

#### Happy Path
1. **Raise Dispute** — Available within 48 hours of inspection result
2. **Dispute Form** → Select issue type: Damage Dispute / Wrong Item / Non-Return / Other → Description → Attach additional photos
3. **Admin receives alert** → opens dispute workspace
4. **Evidence Review** — Admin views: delivery photos vs return photos side-by-side, chat logs, order timeline, agent notes
5. **Admin Decision** → Resolution applied: full refund / partial refund / penalty applied / no action
6. **Both parties notified** → push + email with resolution summary and updated amounts
7. **Payout adjusted** → refund and seller payout recalculated

#### Error States
- Dispute raised outside 48-hour window → "Dispute window has closed for this order."
- Dispute lacks evidence → admin requests additional information from both parties; 24-hour response window

---

## 3. Navigation Map

```
App Root
├── Buyer App
│   ├── Home / Discovery
│   │   ├── Category Browse
│   │   │   └── Product Listing Page (PLP)
│   │   │       └── Product Detail Page (PDP)
│   │   │           └── Checkout Flow
│   │   │               ├── Address Selection
│   │   │               ├── ID Verification (≥₹10K)
│   │   │               ├── Payment
│   │   │               └── Order Confirmation
│   │   ├── Search Results
│   │   └── Curated Collections
│   ├── My Orders
│   │   ├── Active Rentals
│   │   │   ├── Order Detail
│   │   │   │   ├── Schedule Return
│   │   │   │   ├── Chat with Seller
│   │   │   │   ├── Raise Dispute
│   │   │   │   └── Track Delivery
│   │   └── Past Orders
│   │       ├── Order Detail
│   │       └── Leave Review
│   ├── Notifications
│   └── Profile & Account
│       ├── Personal Info
│       ├── Saved Addresses
│       ├── Payment Methods
│       ├── Transaction History
│       ├── Referral Program
│       └── Settings / Logout
│
├── Seller App
│   ├── Dashboard (Home)
│   │   ├── Earnings Summary
│   │   ├── New Orders (Accept/Decline)
│   │   ├── Active Rentals
│   │   └── Alerts & Reminders
│   ├── My Listings
│   │   ├── Add New Listing
│   │   ├── Edit Listing
│   │   ├── Availability Calendar
│   │   └── Bulk Upload (CSV)
│   ├── Orders
│   │   ├── Order Detail
│   │   ├── Raise Dispute
│   │   └── Review Return Photos
│   ├── Analytics
│   │   ├── Views & Conversions
│   │   ├── Revenue Breakdown
│   │   └── Repeat Renters
│   └── Account
│       ├── Store Profile
│       ├── KYC & Verification
│       ├── Bank Details
│       ├── Subscription Plan
│       └── Payout History
│
├── Delivery Agent App (Lightweight PWA)
│   ├── Today's Assignments
│   │   └── Order → Inspection Workflow
│   │       ├── Capture Delivery Photos
│   │       ├── Scan QR Tag
│   │       ├── Buyer OTP Confirmation
│   │       └── Mark Delivered
│   └── Return Pickup
│       ├── Capture Return Photos
│       ├── QR Tag Scan
│       ├── Damage Classification
│       └── Mark Returned
│
└── Admin Panel (Web)
    ├── Dashboard (GMV, Orders, Disputes)
    ├── User Management (Buyers & Sellers)
    ├── Listings (Review, Approve, Reject)
    ├── Orders (Override, Refund, Penalise)
    ├── Disputes (Evidence Viewer, Resolution)
    ├── KYC Review Queue
    ├── Payouts (Trigger, Override, History)
    ├── Marketing (Banners, Collections, Promos)
    └── Settings (Platform Parameters)
```

---

## 4. Screen Inventory

| Screen | Route | Access Level | Purpose | Key Actions | State Variants |
|--------|-------|-------------|---------|-------------|----------------|
| Splash / Onboarding | `/onboarding` | Public | First impression + role selection | Get Started, Role Select | Loading, Animated |
| Buyer Registration | `/register/buyer` | Public | Create buyer account | Enter phone, OTP, password | Default, OTP Active, Error |
| Seller Registration | `/register/seller` | Public | Create seller account | Enter phone, OTP, password | Default, OTP Active, Error |
| Login | `/login` | Public | Authenticate existing user | Phone OTP or password | Default, OTP, Forgot Password |
| Buyer Homepage | `/home` | Authenticated (Buyer) | Discovery + curated collections | Search, Browse, Open listing | Loading skeleton, Empty, Default |
| Category PLP | `/category/:id` | Authenticated (Buyer) | Browse filtered listings | Filter, Sort, Open listing | Loading, Empty, Results |
| Search Results | `/search` | Authenticated (Buyer) | Keyword search results | Filter, Sort, Open listing | Loading, No Results, Results |
| Product Detail (PDP) | `/listing/:id` | Authenticated (Buyer) | Item evaluation + rental calc | Rent Now, Chat, Save | Loading, Available, Unavailable |
| Checkout — Address | `/checkout/address` | Authenticated (Buyer) | Select/add delivery address | Select, Add New | Default, Adding New |
| Checkout — Payment | `/checkout/payment` | Authenticated (Buyer) | Pay for rental | Select method, Pay | Default, Processing, Error, Success |
| Order Confirmation | `/orders/:id/confirmed` | Authenticated (Buyer) | Order success summary | View Order, Go Home | Success state only |
| My Orders | `/orders` | Authenticated (Buyer) | List of all orders | View, Filter by status | Loading, Empty, Active, Past |
| Order Detail | `/orders/:id` | Authenticated (Buyer) | Full order timeline + actions | Schedule Return, Dispute, Chat | Active, Returned, Disputed, Sold |
| Seller Dashboard | `/seller/dashboard` | Authenticated (Seller) | Operations overview | Accept orders, View earnings | Loading, Default, Alert state |
| Add Listing | `/seller/listings/new` | Authenticated (Seller) | Create new rental item | Upload photos, Set price, Publish | Step 1-N wizard |
| Edit Listing | `/seller/listings/:id/edit` | Authenticated (Seller) | Modify existing listing | Update fields, Save | Default, Saved, Error |
| Seller Orders | `/seller/orders` | Authenticated (Seller) | Manage incoming/active orders | Accept, Decline, View | New, Active, Past |
| Dispute Detail | `/disputes/:id` | Auth (Buyer/Seller/Admin) | View evidence and resolution | Submit evidence, View timeline | Open, Under Review, Resolved |
| Admin Dashboard | `/admin` | Admin Only | Platform operations | All admin actions | Default with real-time data |

---

## 5. Decision Points

```
IF user is NOT authenticated
  THEN → redirect to /login with return URL preserved

IF user selects "Buyer"
  THEN → Buyer registration → Buyer Homepage flow

IF user selects "Seller"
  THEN → Seller registration → KYC → Seller Dashboard flow

IF item value ≥ ₹10,000 at checkout
  THEN → require Aadhaar OTP verification before payment
  ELSE → proceed directly to payment

IF seller has ≤ 3 published listings
  THEN → new listing requires admin approval before going live
  ELSE → listing goes live immediately (flagged listings excepted)

IF order day count ≥ 20 and no return scheduled
  THEN → trigger reminder push + SMS; escalate to in-app alert
  ELSE → standard in-app countdown visible

IF order day count ≥ 25 and item not returned
  THEN → auto-convert order to "Sold"; release escrow to seller (minus commission); zero refund to buyer

IF item QR tag scan at return does NOT match delivery scan
  THEN → block return confirmation; alert admin; flag buyer for investigation

IF inspection photos show damage AND seller raises dispute
  THEN → initiate dispute flow; hold refund; admin reviews within 24 hours

IF seller cancellations ≥ 3 in 60 days
  THEN → flag seller account for review; show warning in admin panel

IF delivery agent consistently records "None" damage for returns from same buyer
  THEN → flag buyer-agent pairing for fraud investigation

IF buyer Trustworthiness Score qualifies (5+ rentals, zero damage disputes)
  THEN → offer "Trusted Renter" tier with 60% escrow requirement instead of 100%
```

---

## 6. Error Handling Flows

| Error | Screen Displayed | Available Actions | System Recovery |
|-------|-----------------|-------------------|-----------------|
| **404 — Page Not Found** | "Page not found" illustration + message | Go to Home, Contact Support | Log 404 event |
| **500 — Server Error** | "Something went wrong on our end" + Ref ID | Retry, Contact Support with Ref ID | Auto-alert ops team |
| **Network Offline** | Toast: "No internet connection" | Retry when connected | Queue any write operations |
| **Payment Timeout** | "We're confirming your payment…" spinner | Wait (auto-resolves in 5 min), Contact Support | Webhook retry from Razorpay; refund if unresolved |
| **Session Expired** | Modal: "Your session has expired. Please log in again." | Login → return to previous screen | Preserve navigation intent in URL |
| **Item Already Booked** | "Just rented by someone else" error on checkout | Browse similar items (suggested), Back to search | Release held cart slot |
| **KYC Rejected** | Notification + reason in app | Re-upload corrected documents | Admin review restarts |

---

## 7. Responsive Behaviour

### Mobile (Primary — <768px)
- Bottom navigation bar: Home, Orders, Notifications, Profile (Buyer); Dashboard, Listings, Orders, Profile (Seller)
- PDP photo carousel: full-width swipe; rental calculator collapses into a sticky bottom card
- Checkout: single column, one section per screen
- Listings grid: 2-column card layout
- Filter panel: full-screen bottom sheet slide-up

### Desktop / Tablet (≥768px)
- Top navigation bar replaces bottom nav
- PDP: two-column layout — photos left, details + calculator right
- Checkout: two-column — summary right, form left
- Listings: 3–4 column grid
- Admin panel: sidebar navigation + main content area; only accessible on desktop (≥1024px)
- Filter panel: sidebar inline (not full-screen overlay)
