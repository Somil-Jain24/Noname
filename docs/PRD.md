# Product Requirements Document (PRD)
# RentKart — India's Premier Rental Marketplace
**Version:** 1.0 | **Owner:** Product Team | **Status:** Final Draft — Ready for Engineering

---

## 1. Product Overview

- **Project Title:** RentKart
- **Version:** MVP v1.0 (Phase 1 — Fashion)
- **Owner:** RentKart Product & Engineering Team
- **Target Market:** India (Urban + Semi-Urban)
- **Currency:** INR
- **Platform Priority:** Mobile-First (Android + iOS), Web responsive
- **Primary Category:** Fashion — Bridal, Wedding, Occasion Wear & Accessories

---

## 2. Problem Statement

India hosts over **12 million weddings per year**. The average household spends ₹3–8 lakh on wedding attire that is worn once and rarely used again. Aspiration for premium fashion — designer lehengas, sherwanis, jewellery sets, luxury bags — far outpaces willingness or ability to permanently own these items.

Existing solutions fail users on both sides:

- **Buyers** pay full purchase price for once-worn items, or rely on untrustworthy informal borrowing
- **Sellers/boutiques** sit on expensive idle inventory that depreciates unused between occasions
- **Current rental platforms** (Stage3, Flyrobe) failed due to unresolved trust, hygiene, and fraud problems — no systematic inspection, no financial protection, no accountability

RentKart's escrow-first marketplace model creates a structured, financially protected rental transaction that eliminates deposit ambiguity, prevents fraud, and monetises idle inventory at scale.

---

## 3. Goals & Objectives

### Business Goals
1. **GMV Target:** Achieve ₹50L GMV within 6 months of launch in the first city (Indore/Pune)
2. **Seller Acquisition:** Onboard 25 verified sellers with 150+ live listings before buyer-facing launch
3. **Commission Revenue:** Reach ₹7.5L commission revenue by Month 6 (15% of ₹50L GMV)
4. **Seller Retention:** Maintain ≥85% seller retention rate (sellers completing 2+ rentals/month) in the first year
5. **Category Expansion:** Launch Phase 2 (Electronics) within 18 months of Phase 1 stabilisation

### User Goals
- **Buyers:** Access premium fashion at 2–5% of item value per day, with transparent pricing and guaranteed refunds
- **Sellers:** Monetise idle inventory with zero financial risk — full item value secured in escrow before dispatch
- **Both:** Trust that the platform enforces fair, transparent rules without bias

---

## 4. Success Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Checkout Conversion Rate | ≥30% (browse → order) | Analytics |
| Cart Abandonment Rate | <40% at escrow step | Funnel tracking |
| Dispute Rate | <5% of completed orders | Dispute tickets |
| Seller Net Promoter Score | ≥50 | Monthly NPS survey |
| Avg. Order Return Rate (non-abuse) | ≥92% returned within 25 days | Order data |
| Time to Refund Processing | <7 business days | Payout logs |

---

## 5. Target Users & Personas

### Persona 1 — Priya, The Event Renter (Buyer)
- **Age/Demographics:** 24–34, urban or semi-urban, working professional or college student
- **Location:** Tier 1–2 Indian cities (Indore, Pune, Jaipur, Surat)
- **Occasion:** Wedding attendee, college fest performer, photoshoot participant
- **Pain Points:** Can't justify spending ₹25,000 on a lehenga worn once; borrowing from friends is awkward; no trusted platform
- **Goals:** Wear premium fashion for an event at a fraction of purchase price; get refund quickly afterward
- **Tech Proficiency:** High — uses Swiggy, Meesho, Myntra daily; comfortable with UPI
- **Trust Concern:** "Will I actually get my money back? What if the item arrives damaged?"

### Persona 2 — Sunita Boutique Owner (Seller)
- **Age/Demographics:** 35–50, runs a fashion boutique in a wedding market city
- **Business Profile:** Has 20–100 high-value items (bridal lehengas ₹15,000–₹80,000) with 60–70% idle time
- **Pain Points:** Inventory sits unused for months between rentals; existing informal rental customers don't pay on time; no financial protection against damage
- **Goals:** Generate passive income on idle stock; reach more customers beyond local market; reduce risk of buyers keeping or damaging items
- **Tech Proficiency:** Medium — uses WhatsApp Business, Instagram, basic Android apps
- **Trust Concern:** "What if my ₹60,000 lehenga doesn't come back? What if the buyer damages it?"

---

## 6. Features & Requirements

### P0 — Must Have (MVP)

---

#### Feature 1: Escrow-First Payment System
**Description:** Buyer pays full item value + delivery charge at checkout. Platform holds item value in escrow. After confirmed return and inspection, refund is issued (item value minus usage charges, damage penalties, and late fees).

**User Story:** "As a buyer, I want to see exactly how much I'll pay and how much I'll get back before I place the order, so I have no surprises at checkout or refund."

**Acceptance Criteria:**
- [ ] Checkout displays: escrow amount, delivery charge, estimated usage fee, and expected refund range — all before payment confirmation
- [ ] No hidden fees revealed post-payment; all deductions must be pre-disclosed
- [ ] Rental calculator on Product Detail Page (PDP) shows: rental days, daily cost, total usage charge, expected refund
- [ ] Escrow funds held in segregated account via Razorpay; never used for operational expenses
- [ ] Refund credited to original payment method within 5–7 business days of inspection pass
- [ ] Checkout labels escrow as "Refundable Security Hold" (not "deposit" or bare item value)

**Success Metric:** Cart abandonment at payment step < 40%

---

#### Feature 2: Product Listing & Catalog
**Description:** Sellers can create, manage, and publish listings with photos, pricing, condition, size, and availability. Buyers can browse, search, and filter the catalog.

**User Story:** "As a seller, I want to list my bridal lehenga with photos, set my daily rental rate, and publish it live — so buyers can find and rent it."

**Acceptance Criteria:**
- [ ] Required listing fields: Title (≤80 chars), Description (≥100 chars), Min 4 photos (front/back/detail/flat-lay), Category + subcategory, Item Value (₹), Daily Rental % (0.5%–5%), Condition, Availability calendar, Location/pincode
- [ ] Optional fields: Size/fit, colour, occasion tags, care instructions, bundle combos, brand name
- [ ] Seller can set blackout dates (dates unavailable)
- [ ] Calendar auto-blocks confirmed booking dates
- [ ] Buyer search supports English and Hindi keywords; fuzzy search for typos (e.g., "lehanga" → "lehenga")
- [ ] Filter by: Category, City/Pincode, Size, Price range, Condition, Rating, Availability dates
- [ ] Sort by: Relevance, Price (low to high), Rating, Newest, Distance
- [ ] Admin must approve first 3 listings per new seller; subsequent listings go live immediately

**Success Metric:** 150+ live listings at launch; avg 4+ photos per listing

---

#### Feature 3: Order Management & Buyer Journey
**Description:** End-to-end order flow from discovery to refund — covering checkout, delivery scheduling, in-use tracking, return scheduling, and post-inspection refund.

**User Story:** "As a buyer, I want to track my order from payment through delivery, use the item, schedule a return, and receive my refund automatically — without having to chase anyone."

**Acceptance Criteria:**
- [ ] Buyer receives order confirmation with: Order ID, expected delivery date, rental policy summary, and countdown to Day 25
- [ ] Real-time order status visible: Confirmed → Dispatched → Delivered → In Use → Return Scheduled → Returned → Inspected → Refund Issued
- [ ] Buyer can schedule return at any time during rental window
- [ ] System recalculates usage charge based on actual days used (delivery date to pickup date, inclusive)
- [ ] Push + SMS reminder sent on Day 20, Day 23, and Day 24 if not returned
- [ ] On Day 25 with no return: order status auto-converts to "Sold"; full item value released to seller minus commission; zero refund to buyer
- [ ] Buyer can view real-time "current usage charges accrued" during rental
- [ ] After return, buyer is prompted (push + email) to rate item and seller

**Success Metric:** ≥92% of orders returned within 25 days; refund issued within 7 business days

---

#### Feature 4: Delivery & Photo Inspection System
**Description:** Platform-managed or seller self-delivery with mandatory in-app photo/video inspection at both delivery and return. Photos are timestamped, GPS-tagged, and stored against each order.

**User Story:** "As a seller, I want photo evidence captured at delivery and return, so that damage disputes are resolved fairly and I'm not left arguing with a buyer."

**Acceptance Criteria:**
- [ ] Mandatory in-app photo capture (no gallery uploads) at delivery: minimum 8 photos from fixed angles guided by in-app overlay
- [ ] Return inspection: same 8-angle process + agent notes any damage with written description
- [ ] All photos: EXIF + GPS + timestamp locked at capture
- [ ] Buyer provides in-app OTP or e-signature to confirm receipt at delivery
- [ ] Seller has 24-hour review window after return photos are uploaded before platform releases refund
- [ ] Delivery agent can flag damage level: None / Minor / Moderate / Severe
- [ ] For items ≥₹10,000: QR-coded item tag sewn by seller must be scanned at delivery and return — mismatch blocks confirmation
- [ ] For platform-managed logistics: delivery agent app (lightweight PWA) displays assigned orders, addresses, and inspection workflow

**Success Metric:** Dispute rate < 5%; photo completeness ≥ 95% of orders

---

#### Feature 5: Seller Dashboard & Onboarding
**Description:** Seller onboarding flow (KYC, bank details, store profile) and ongoing dashboard (earnings, order management, analytics, availability calendar).

**User Story:** "As a seller, I want to see my earnings, manage my listings, and track upcoming orders — all in one place without needing to call anyone."

**Acceptance Criteria:**
- [ ] Onboarding: Phone OTP → KYC upload (Aadhaar/PAN + address proof) → Bank details (penny drop verification) → Store profile → First listing
- [ ] Admin reviews KYC within 1 business day
- [ ] Dashboard shows: earnings this month, total rentals, pending payout, avg rating
- [ ] Seller can accept/decline orders within 2-hour window
- [ ] Seller can: bulk upload via CSV (for 10+ items), mark items "Under Cleaning" (removes from availability), set promotional rental rates for date ranges
- [ ] Seller payout: Usage charge minus 15% commission; paid via NEFT/IMPS within 7 business days of inspection pass
- [ ] Seller can choose: self-delivery or platform-managed logistics

**Success Metric:** Seller onboarding completion rate ≥ 75%; payout SLA met ≥ 95% of the time

---

#### Feature 6: Trust & Verification System
**Description:** Identity verification for buyers and sellers, Trustworthiness Score, fraud detection (device fingerprinting, contact masking), and review system.

**User Story:** "As a buyer, I want to know the seller is verified and legitimate — and as a seller, I want to know the buyer is trustworthy — before any item changes hands."

**Acceptance Criteria:**
- [ ] Buyer: Phone OTP mandatory; ID proof (Aadhaar-linked OTP via DigiLocker/eKYC) required for orders ≥₹10,000
- [ ] Seller: Phone OTP + Aadhaar or PAN + address proof mandatory before first listing
- [ ] In-app chat: all messages pass through platform proxy; phone numbers, UPI handles, social media links auto-redacted
- [ ] Contact info never exposed in UI (seller number masked behind order-linked temporary numbers)
- [ ] Device fingerprinting: links new accounts to previously suspended devices
- [ ] Reviews only allowed for verified completed orders; buyer must confirm receipt first
- [ ] Sellers with ≥3 cancellations in 60 days are flagged for review
- [ ] Agents with consistent "None" damage ratings from same buyer are flagged for investigation

**Success Metric:** 0 verified KYC fraud cases in first 6 months; < 2% off-platform bypass reports

---

#### Feature 7: Dispute Resolution System
**Description:** Structured dispute workflow where buyers or sellers can raise a claim; admin reviews evidence (photos, chat logs, delivery records) and issues a resolution within defined SLA.

**User Story:** "As a buyer, I want to raise a dispute if I'm charged for damage I didn't cause — and get a fair, timely resolution from the platform."

**Acceptance Criteria:**
- [ ] Buyer or seller can raise a dispute from the order detail page within 48 hours of inspection result
- [ ] Admin dispute workspace: evidence viewer (delivery vs return photos side-by-side), chat logs, order timeline
- [ ] Standard dispute SLA: resolved within 72 hours
- [ ] High-stakes dispute (damage claim ≥₹1,000 or refund impact ≥₹5,000): SLA 24 hours
- [ ] Admin can: trigger manual refund, partial refund, penalty charge, or override any order state
- [ ] Both parties notified of resolution outcome via push + email
- [ ] Dispute outcomes logged for pattern analysis

**Success Metric:** 100% of disputes resolved within SLA; < 2% of resolutions re-disputed

---

#### Feature 8: Admin Panel
**Description:** Internal operations dashboard for managing users, listings, orders, disputes, payouts, KYC, platform parameters, and analytics.

**User Story:** "As a platform admin, I want to approve listings, resolve disputes, and monitor GMV — all from a single dashboard without needing to access the database."

**Acceptance Criteria:**
- [ ] View, verify, block, or flag any buyer/seller account
- [ ] Review and approve/reject KYC documents and new listings
- [ ] Full dispute resolution workspace with evidence viewer
- [ ] Manual refund, partial refund, and penalty charge triggers
- [ ] Platform-wide analytics: GMV, commissions, payout summaries, daily active users, order volume
- [ ] Manage homepage banners, featured collections, and promo campaigns
- [ ] Manage seller subscription billing; override plan status
- [ ] Set platform-wide parameters (e.g., non-return auto-sale day, commission %)
- [ ] Blacklist fraudulent users and log abuse incidents with reason codes

**Success Metric:** Admin panel handles 100% of operational tasks without direct DB access

---

### P1 — Should Have (Post-MVP Priority)

- **Referral Program:** ₹200 credit per referred user who completes their first rental; device fingerprint check to prevent self-referral; max ₹1,000 credit per account per 90 days; credits apply only to rental fees (not escrow)
- **Trustworthiness Score:** Accumulated score for both buyers and sellers based on on-time returns, damage history, completion rate; unlocks reduced escrow for "Trusted Renter" tier (5+ successful rentals, zero damage disputes → 60% escrow instead of 100%)
- **Seller Subscription Plan (₹299/month):** Commission 12% vs 15%, priority ranking, advanced analytics, bulk CSV upload, "Verified Premium Seller" badge, 2-hour support SLA
- **Featured Listing Boosts:** ₹99–499/listing for 7-day rank boost in search and category pages
- **Occasion-Based Bundles:** Sellers can create rental bundle (lehenga + jewellery + clutch) as single rental package with one checkout and one return
- **Rating & Review System with Photos:** 1–5 star, written review, and 1–5 photos; mandatory for ≤4 stars; "Verified Rental" badge; seller 24-hour flag window

### P2 — Nice to Have

- **Rent-to-Own Toggle:** Buyer indicates interest in purchasing at checkout; if they keep item (Day 25), rental fees paid are credited toward purchase price
- **Rental Insurance Integration:** Optional ₹50 add-on covers accidental damage up to ₹2,000; reduces buyer anxiety about penalties
- **Reverse Discovery ("Available For"):** Sellers post available date ranges; buyers searching a specific date see available items
- **AI-Assisted Damage Detection:** Phase 2 upgrade — computer vision on return photos flags potential damage before admin review
- **Seller Concierge Onboarding:** Free professional photo shoot for first 50 sellers' top 5 listings
- **Tier 2 City Expansion:** Surat, Jaipur, Coimbatore — Phase 2 geographic rollout

---

## 7. Explicitly OUT OF SCOPE

1. **Phase 2 / 3 Categories (Electronics, Camping, Furniture, Party Supplies):** Not built in MVP — Phase 1 is fashion only
2. **International Markets / Non-INR Currencies:** India-only at launch
3. **Own Inventory Model:** RentKart does not purchase or own inventory; 100% marketplace model
4. **B2B / Corporate Rental:** No bulk enterprise orders or corporate accounts in MVP
5. **In-App Auction or Bidding:** No dynamic pricing auctions; all pricing is seller-set fixed daily rates
6. **Social Feed / Community Forum:** No Instagram-style browsing feed or community chat
7. **Subscription Box / Curated Delivery Service:** No recurring scheduled deliveries
8. **Live Video Try-On or AR Features:** No augmented reality or live streaming features in MVP
9. **Seller Marketplace Ads (Google/Meta Integration):** No external ad buying platform integration
10. **Multi-Language UI Beyond Hindi/English:** Only Hindi and English supported at launch
11. **Seller Financing or Advances Against Inventory:** No lending or financial products for sellers
12. **Dry-Cleaning Pickup Service (Platform-Operated):** Platform may partner but does not own or operate cleaning operations

---

## 8. User Scenarios

### Scenario 1 — First-Time Buyer (Happy Path)
**Context:** Priya needs a bridal lehenga for her cousin's wedding in Pune; budget ₹600/day.

1. Opens RentKart app → sees "Wedding Season Picks" on homepage
2. Searches "bridal lehenga Pune" → filters: Size M, Budget ₹500–800/day, Rating ≥4 stars
3. Opens listing → views 8 photos → checks seller rating (4.7 ★, 23 rentals)
4. Uses Rental Calculator: 5-day rental of ₹20,000 item @ 3% → ₹600/day, ₹3,000 usage charge, ₹17,000 refund expected
5. Adds to cart → sees full breakdown (₹20,150 checkout total: ₹20,000 escrow + ₹150 delivery)
6. Pays via UPI → receives order confirmation SMS + push
7. Receives item with delivery agent → opens inspection confirmation screen → photos captured
8. Wears to wedding → returns on Day 4 → agent picks up, captures return photos
9. Inspection pass → ₹17,000 refund triggered → credited in 6 days
10. Priya rates item 5 stars with photos

**Edge Cases:** Seller rejects order (auto-refund + notifications; buyer must re-select); item arrives in wrong condition (dispute raised immediately at delivery); item not returned by Day 24 (platform sends escalation notification)

---

### Scenario 2 — Seller Listing & First Rental
**Context:** Sunita's boutique has 12 premium lehengas sitting idle.

1. Downloads app → registers with phone OTP → uploads KYC (Aadhaar + address proof)
2. Admin approves KYC within 24 hours
3. Creates first listing: 8 photos uploaded, item value ₹40,000, daily rate 2%, condition "Like New"
4. Admin reviews and approves first listing within 4 hours
5. Receives first order notification (2-hour window to accept)
6. Accepts order → marks item "Ready to Dispatch"
7. Platform delivery agent arrives → scans QR tag on lehenga → delivers to buyer
8. After return and inspection pass → receives payout of ₹680 (₹800 usage - ₹120 commission) within 7 days

**Edge Cases:** Buyer disputes damage (Sunita reviews return photos in 24-hour window; if disagreement, admin decides based on evidence); item not returned by Day 25 (platform auto-converts to sale, Sunita receives ₹34,000)

---

### Scenario 3 — Dispute Resolution
**Context:** Buyer returns item; seller claims damage (torn embroidery); buyer denies causing it.

1. Seller raises dispute within 48 hours via dashboard → attaches delivery vs return photo comparison
2. Admin opens dispute workspace → views side-by-side photo comparison (delivery vs return)
3. Admin reviews: damage visible in return photo but NOT in delivery photo → confirms damage post-rental
4. Damage classified as "Moderate" → ₹2,000 penalty applied to buyer
5. Buyer refund: item value ₹25,000 – ₹2,500 usage charge – ₹2,000 damage = ₹20,500
6. Seller payout: ₹2,500 usage – ₹375 commission + ₹2,000 damage penalty forwarded = ₹4,125
7. Both notified of resolution within 72-hour SLA

**Edge Cases:** Photos inconclusive → admin requests delivery agent written notes; buyer accuses staged photos → seller damage rate flagged for review

---

## 9. Dependencies & Constraints

### Technical Constraints
- Must integrate with Razorpay for escrow-safe payment processing (PA-licensed partner)
- Must support UPI, cards, net banking, and wallets at minimum
- Photo capture must be in-app only (no gallery upload for inspection) — requires native camera API
- Device fingerprinting requires native mobile SDK (cannot be web-only)

### Business Constraints
- Escrow funds must be held in segregated account — never co-mingled with operational funds
- KYC/eKYC integration requires DigiLocker or RBI-approved eKYC partner
- GST compliance via ClearTax or equivalent tax engine
- Seller payout must support NEFT/IMPS to Indian bank accounts

### External Dependencies
- Razorpay — Payment gateway + escrow
- Delhivery / Shiprocket — Platform logistics (3PL option)
- DigiLocker / eKYC provider — Identity verification
- ClearTax or Tax-Engine API — GST computation
- Firebase / OneSignal — Push notifications

---

## 10. Timeline & Milestones

| Milestone | Target Date | Features Included |
|-----------|-------------|-------------------|
| **Foundation** | Week 2 | Project setup, DB schema, design tokens |
| **Alpha (Internal)** | Week 5 | Auth, listings, admin KYC review |
| **Seller Beta** | Week 8 | Full seller onboarding, listing management, escrow checkout |
| **MVP Launch** | Week 12 | All P0 features, 25 sellers onboarded, 150+ listings live |
| **V1.1** | Month 5 | Referral program, Trustworthiness Score, seller subscription |
| **V2.0** | Month 12 | Phase 2 categories (Electronics), Tier 2 city expansion |

---

## 11. Non-Functional Requirements

### Performance
- Page load time < 2 seconds on 4G mobile connection
- Checkout API response < 1 second
- Photo upload (8 images) < 15 seconds on 4G

### Security
- All payment data handled by Razorpay; RentKart never stores raw card data (PCI-DSS compliant via gateway)
- Aadhaar/PAN data encrypted at rest (AES-256) and in transit (TLS 1.3)
- JWT access tokens expire in 15 minutes; refresh tokens in 30 days
- bcrypt with 12 rounds for password hashing
- Rate limiting: login — 5 attempts/15 min/IP; registration — 10/hour; API (public) — 100/min; API (authenticated) — 500/min

### Accessibility
- WCAG 2.1 Level AA compliance for web
- Minimum touch target: 44×44px
- All interactive elements keyboard-navigable

### Scalability
- Infrastructure must support 10× current load without architecture changes
- Database reads/writes separated (read replica) from Month 3 onward

### Availability
- Target uptime: 99.5% monthly
- Planned maintenance windows communicated 24h in advance
